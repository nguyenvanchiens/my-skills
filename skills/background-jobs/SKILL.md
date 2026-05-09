---
name: background-jobs
description: "Background job patterns trong ASP.NET Core: `IHostedService`/`BackgroundService` (built-in cho queue/scheduled), `System.Threading.Channels` (in-memory producer-consumer), Hangfire (persistent fire-and-forget/scheduled/recurring với dashboard), Quartz.NET (cron-based scheduling phức tạp). Use khi cần process async sau request response, scheduled task (daily report, cleanup), retry với exponential backoff, queue work để giảm response latency, hoặc user nói 'send email async', 'schedule task', 'cron job', 'queue consumer'."
compatibility: "Built-in `IHostedService` cần ASP.NET Core. Hangfire cần persistence (SQL Server/PostgreSQL/Redis). Quartz cần persistence cho cluster mode."
---

# Background Jobs trong ASP.NET Core

## Trigger On

- "send email async sau khi user submit form"
- "scheduled task chạy daily/hourly" (cleanup, report, sync)
- "retry job khi failure", "exponential backoff"
- "queue consumer" (process messages từ Channel/RabbitMQ/Service Bus)
- chọn giữa `BackgroundService`, Hangfire, Quartz, MassTransit
- offload work khỏi request thread để giảm latency response

## Documentation

- [BackgroundService / IHostedService](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services)
- [System.Threading.Channels](https://learn.microsoft.com/en-us/dotnet/core/extensions/channels)
- [Hangfire](https://www.hangfire.io/)
- [Quartz.NET](https://www.quartz-scheduler.net/)

## Quick Decision

```
Long-running consumer trong process app           → BackgroundService
Producer-consumer trong process (in-memory queue) → BackgroundService + Channel<T>
Persistent job (survive restart), retry, dashboard → Hangfire
Cron schedule phức tạp, cluster scheduling        → Quartz.NET
Cross-service queue (microservices)               → MassTransit / RabbitMQ / Service Bus
```

## Pattern 1: `BackgroundService` — built-in

Dùng khi: long-running task lifetime gắn với app (consumer queue, periodic poll, scheduled task đơn giản).

```csharp
public class CleanupService(
    IServiceScopeFactory scopeFactory,
    ILogger<CleanupService> logger,
    TimeProvider time) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        using var timer = new PeriodicTimer(TimeSpan.FromHours(1), time);

        while (!ct.IsCancellationRequested)
        {
            try
            {
                await using var scope = scopeFactory.CreateAsyncScope();
                var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

                var cutoff = time.GetUtcNow().AddDays(-30);
                var deleted = await db.AuditLogs
                    .Where(a => a.CreatedAt < cutoff)
                    .ExecuteDeleteAsync(ct);

                logger.LogInformation("Deleted {Count} audit logs", deleted);
            }
            catch (Exception ex) when (ex is not OperationCanceledException)
            {
                logger.LogError(ex, "Cleanup failed");
            }

            try { await timer.WaitForNextTickAsync(ct); }
            catch (OperationCanceledException) { break; }
        }
    }
}

// Register
builder.Services.AddHostedService<CleanupService>();
```

> **Quan trọng**:
> - `BackgroundService` là **Singleton** → KHÔNG inject scoped service (DbContext) trực tiếp. Dùng `IServiceScopeFactory.CreateAsyncScope()` per iteration.
> - Wrap loop trong try/catch — exception unhandled sẽ kill service, không tự restart.
> - Honor `CancellationToken` để graceful shutdown.

## Pattern 2: `BackgroundService` + `Channel<T>` — in-memory queue

Dùng khi: producer (controller/service) push work, consumer process async, không cần persistence.

```csharp
public record EmailJob(string To, string Subject, string Body);

// Singleton queue wrapper
public class EmailQueue
{
    private readonly Channel<EmailJob> _channel = Channel.CreateBounded<EmailJob>(
        new BoundedChannelOptions(1000)
        {
            FullMode = BoundedChannelFullMode.Wait
        });

    public ChannelReader<EmailJob> Reader => _channel.Reader;
    public ValueTask EnqueueAsync(EmailJob job, CancellationToken ct = default)
        => _channel.Writer.WriteAsync(job, ct);
}

// Consumer
public class EmailConsumer(
    EmailQueue queue,
    IServiceScopeFactory scopeFactory,
    ILogger<EmailConsumer> logger) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken ct)
    {
        await foreach (var job in queue.Reader.ReadAllAsync(ct))
        {
            try
            {
                await using var scope = scopeFactory.CreateAsyncScope();
                var sender = scope.ServiceProvider.GetRequiredService<IEmailSender>();
                await sender.SendAsync(job.To, job.Subject, job.Body, ct);
            }
            catch (Exception ex)
            {
                logger.LogError(ex, "Failed to send email to {To}", job.To);
            }
        }
    }
}

// Register
builder.Services.AddSingleton<EmailQueue>();
builder.Services.AddHostedService<EmailConsumer>();

// Producer (controller)
public class OrderController(EmailQueue queue) : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> Place(OrderDto dto)
    {
        var order = await CreateOrderAsync(dto);
        await queue.EnqueueAsync(new EmailJob(order.Email, "Order Placed", order.Summary));
        return Ok(order);
    }
}
```

> ⚠️ In-memory queue — **mất khi app restart/crash**. Production cần persistent queue (Hangfire, Service Bus).

## Pattern 3: Hangfire — persistent jobs với dashboard

Dùng khi: cần job survive restart, retry tự động, scheduled/recurring với UI dashboard, không muốn build infra queue.

```bash
dotnet add package Hangfire.AspNetCore
dotnet add package Hangfire.SqlServer       # hoặc Hangfire.PostgreSql, Hangfire.Redis
```

```csharp
builder.Services.AddHangfire(config => config
    .SetDataCompatibilityLevel(CompatibilityLevel.Version_180)
    .UseSimpleAssemblyNameTypeSerializer()
    .UseRecommendedSerializerSettings()
    .UseSqlServerStorage(builder.Configuration.GetConnectionString("Hangfire")));

builder.Services.AddHangfireServer(opt =>
{
    opt.WorkerCount = Environment.ProcessorCount * 2;
    opt.Queues = ["default", "emails", "reports"];
});

var app = builder.Build();

app.UseHangfireDashboard("/hangfire", new DashboardOptions
{
    Authorization = [new HangfireAuthorizationFilter()]   // protect dashboard!
});
```

### Job types

```csharp
// 1. Fire-and-forget (run once, asap)
BackgroundJob.Enqueue<IEmailService>(s => s.SendAsync(to, subject, body));

// 2. Delayed
BackgroundJob.Schedule<IEmailService>(
    s => s.SendAsync(to, subject, body),
    TimeSpan.FromMinutes(30));

// 3. Recurring (cron)
RecurringJob.AddOrUpdate<ICleanupService>(
    "cleanup-old-logs",
    s => s.CleanupAsync(),
    Cron.Daily(2));   // 2am every day

// 4. Continuation (chain after another job)
var jobId = BackgroundJob.Enqueue<IOrderService>(s => s.ProcessAsync(orderId));
BackgroundJob.ContinueJobWith<IEmailService>(
    jobId,
    s => s.SendConfirmationAsync(orderId));
```

### Job class — recommended

```csharp
public interface IEmailService
{
    Task SendAsync(string to, string subject, string body);
}

// Hangfire instantiate qua DI scope per job execution
public class EmailService(SmtpClient smtp, AppDbContext db, ILogger<EmailService> logger) : IEmailService
{
    [AutomaticRetry(Attempts = 5, DelaysInSeconds = [60, 300, 900, 3600, 21600])]
    public async Task SendAsync(string to, string subject, string body)
    {
        await smtp.SendMailAsync(...);
        await db.EmailLogs.AddAsync(new EmailLog { To = to, SentAt = DateTime.UtcNow });
        await db.SaveChangesAsync();
    }
}
```

> **Hangfire pros**: dashboard UI, persistent, retry/continuation built-in, well-documented. **Cons**: thêm DB tables (Hangfire schema), license LGPL cho free version (Pro version cho enterprise).

## Pattern 4: Quartz.NET — cron scheduling phức tạp

Dùng khi: lịch chạy phức tạp (calendar exclusions, misfire policy, cluster), không cần dashboard nặng như Hangfire.

```bash
dotnet add package Quartz.Extensions.Hosting
```

```csharp
builder.Services.AddQuartz(q =>
{
    q.UseMicrosoftDependencyInjectionJobFactory();

    var jobKey = new JobKey("DailyReport");
    q.AddJob<DailyReportJob>(opts => opts.WithIdentity(jobKey));
    q.AddTrigger(opts => opts
        .ForJob(jobKey)
        .WithIdentity("DailyReport-trigger")
        .WithCronSchedule("0 0 9 * * ?"));   // 9am every day
});

builder.Services.AddQuartzHostedService(opt => opt.WaitForJobsToComplete = true);
```

```csharp
public class DailyReportJob(IReportService report, ILogger<DailyReportJob> logger) : IJob
{
    public async Task Execute(IJobExecutionContext context)
    {
        try
        {
            await report.GenerateDailyAsync(context.CancellationToken);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Daily report failed");
            throw new JobExecutionException(ex, refireImmediately: false);
        }
    }
}
```

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| Inject scoped service vào `BackgroundService` ctor | Service Singleton, scoped không match | `IServiceScopeFactory.CreateAsyncScope()` per iteration |
| Quên handle exception trong `ExecuteAsync` | Service crash, không tự restart | Try/catch trong loop |
| Loop không respect `CancellationToken` | Graceful shutdown hang | `await ... ct`, break khi `IsCancellationRequested` |
| In-memory queue cho job critical | Mất khi restart | Hangfire / persistent queue |
| Long-running task `Task.Run(async () => ...)` trong controller | Không track, không retry, leak | Background service / queue |
| Hangfire dashboard không auth | Public access kill jobs / xem nhạy cảm | `IDashboardAuthorizationFilter` |
| `Thread.Sleep` trong loop | Block thread pool | `await Task.Delay(... , ct)` hoặc `PeriodicTimer` |
| Process job >1 hour mà không checkpoint | Restart mất hết progress | Chia job thành chunks, ghi state |
| Recurring job collision (cùng method, cùng time, multi-server) | Chạy duplicate | Hangfire/Quartz dùng distributed lock; hoặc chỉ chạy 1 instance |
| Synchronous I/O trong job | Block worker, giảm throughput | Async everywhere |

## Performance

- **`Channel.CreateBounded`** thay `CreateUnbounded` để limit memory
- **Worker count**: Hangfire mặc định = `Environment.ProcessorCount * 5`. Tune theo workload (CPU-bound vs I/O-bound)
- **Batch processing**: process 100 jobs/scope thay vì 1 job/scope cho job nhỏ
- **Hangfire queue priority**: assign job vào queue khác nhau, multiple HangfireServer mỗi server consume queue khác

## Validate

- Đúng pattern theo nhu cầu (decision matrix trên)
- `BackgroundService` dùng `IServiceScopeFactory` cho scoped deps
- Loop có try/catch + honor `CancellationToken`
- Hangfire/Quartz có persistent storage trong production
- Hangfire dashboard có auth filter
- Recurring job idempotent (chạy 2 lần không gây lỗi)
- Long-running jobs có checkpoint/resume capability
- Logging structured cho mọi job execution (skill `aspnet-logging`)
- Health check verify worker active (skill `aspnet-health-checks`)

## Hand off to

- Logging job events → `aspnet-logging`
- Health check worker status → `aspnet-health-checks`
- DB transactions trong job → `entity-framework-core`
- Caching kết quả job → `aspnet-caching`
