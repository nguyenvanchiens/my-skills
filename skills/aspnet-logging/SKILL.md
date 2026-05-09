---
name: aspnet-logging
description: "Structured logging trong ASP.NET Core với built-in `ILogger<T>` và Serilog: sinks (Console/File/Seq/Elasticsearch), enrichment (correlation ID, user context, environment), log scopes, log levels per namespace, request logging middleware, sensitive data redaction. Use khi setup logging cho production app, debug logs scattered/khó truy vết, cần distributed tracing, output bị flat string không structured, hoặc khi user nói 'logs khó tìm', 'cần track request flow', 'log chậm', 'thêm Serilog'."
compatibility: "Requires ASP.NET Core project. Recommended Serilog cho production (built-in logging đủ cho dev/simple apps)."
---

# ASP.NET Core Structured Logging

## Trigger On

- setup logging cho app mới hoặc upgrade từ Console.WriteLine/text logs
- thêm Serilog, Seq, Elasticsearch, Application Insights sinks
- enrich logs với correlation ID, user context, request path
- request/response logging middleware
- redact sensitive data (password, token, PII) khỏi logs
- log levels per namespace (`Microsoft.EntityFrameworkCore` debug, app info)
- distributed tracing với OpenTelemetry

## Documentation

- [Logging in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging)
- [Serilog](https://serilog.net/)
- [Serilog.AspNetCore](https://github.com/serilog/serilog-aspnetcore)
- [OpenTelemetry .NET](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel)

## Built-in `ILogger` vs Serilog — khi nào dùng cái nào

| | Built-in `ILogger<T>` | Serilog |
|---|---|---|
| Setup | Có sẵn, không cần package | `Serilog.AspNetCore` |
| Structured logging | ✓ (qua message template) | ✓ (richer, easier sinks) |
| Sinks | Console, Debug, EventSource, EventLog | 100+ (File, Seq, Elastic, Application Insights, Splunk...) |
| Enrichment | Limited (Scopes) | Rich (`UseSerilog().Enrich.WithX()`) |
| Recommend | Dev, simple app, library | **Production app**, multi-sink, enrichment |

> **Rule chung**: dev / library code → built-in. Production app → Serilog. App ASP.NET Core hosted → Serilog kết hợp `WriteTo.Console()` + sink production (Seq/Elastic/AppInsights).

## Structured Logging — message template, KHÔNG string interpolation

```csharp
// ❌ Bad — string interpolation, không structured (sink chỉ thấy 1 string)
_logger.LogInformation($"User {userId} purchased {productId} at ${price}");

// ✅ Good — message template, structured (sink lưu userId/productId/price riêng)
_logger.LogInformation(
    "User {UserId} purchased {ProductId} at {Price:C}",
    userId, productId, price);
```

> Properties trong template (PascalCase: `{UserId}`, `{ProductId}`) → tự thành searchable fields trong sink. Interpolation `$""` mất hết structure.

## Setup Serilog

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Sinks.Seq          # nếu dùng Seq local
dotnet add package Serilog.Enrichers.Environment
dotnet add package Serilog.Enrichers.Process
dotnet add package Serilog.Enrichers.Thread
```

```csharp
// Program.cs
using Serilog;

var builder = WebApplication.CreateBuilder(args);

builder.Host.UseSerilog((ctx, services, config) => config
    .ReadFrom.Configuration(ctx.Configuration)
    .ReadFrom.Services(services)
    .Enrich.FromLogContext()
    .Enrich.WithEnvironmentName()
    .Enrich.WithMachineName()
    .Enrich.WithProperty("Application", "MyApp")
    .WriteTo.Console(
        outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj} {Properties:j}{NewLine}{Exception}")
    .WriteTo.File("logs/app-.log",
        rollingInterval: RollingInterval.Day,
        retainedFileCountLimit: 14)
    .WriteTo.Seq("http://localhost:5341"));

var app = builder.Build();

app.UseSerilogRequestLogging();   // log mỗi HTTP request

app.Run();
```

```json
// appsettings.json — config bằng file, dễ thay đổi không recompile
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft.AspNetCore": "Warning",
        "Microsoft.EntityFrameworkCore": "Warning",
        "Microsoft.EntityFrameworkCore.Database.Command": "Information"
      }
    },
    "WriteTo": [
      { "Name": "Console" },
      {
        "Name": "Seq",
        "Args": { "serverUrl": "http://localhost:5341" }
      }
    ],
    "Enrich": ["FromLogContext", "WithEnvironmentName", "WithMachineName"]
  }
}
```

## Correlation ID + Request Logging

### Auto correlation qua middleware

```csharp
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;
    private const string HeaderName = "X-Correlation-ID";

    public CorrelationIdMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context, IDiagnosticContext diagnostics)
    {
        var correlationId = context.Request.Headers[HeaderName].FirstOrDefault()
                            ?? Guid.NewGuid().ToString();

        context.Response.Headers[HeaderName] = correlationId;

        // Thêm vào Serilog LogContext → mọi log trong request này tự có CorrelationId
        using (Serilog.Context.LogContext.PushProperty("CorrelationId", correlationId))
        {
            diagnostics.Set("CorrelationId", correlationId);
            await _next(context);
        }
    }
}

// Program.cs — đặt SỚM trong pipeline
app.UseMiddleware<CorrelationIdMiddleware>();
app.UseSerilogRequestLogging();
```

### Customize Serilog request logging

```csharp
app.UseSerilogRequestLogging(options =>
{
    options.MessageTemplate = "HTTP {RequestMethod} {RequestPath} → {StatusCode} in {Elapsed:0.0}ms";

    options.GetLevel = (httpContext, elapsed, ex) => ex != null
        ? LogEventLevel.Error
        : httpContext.Response.StatusCode >= 500 ? LogEventLevel.Error
        : httpContext.Response.StatusCode >= 400 ? LogEventLevel.Warning
        : elapsed > 1000 ? LogEventLevel.Warning
        : LogEventLevel.Information;

    options.EnrichDiagnosticContext = (diagnostics, httpContext) =>
    {
        diagnostics.Set("UserId", httpContext.User.FindFirst("sub")?.Value);
        diagnostics.Set("RequestHost", httpContext.Request.Host.Value);
        diagnostics.Set("UserAgent", httpContext.Request.Headers.UserAgent.ToString());
    };
});
```

## Log Levels — guideline

| Level | Khi dùng | Ví dụ |
|---|---|---|
| `Trace` | Chi tiết step-by-step (loop iteration) | "Processing item 47/1000" — chỉ on-demand |
| `Debug` | Dev debug info | "Cache miss for key {Key}", SQL query EF |
| `Information` | Sự kiện quan trọng app flow | "User {UserId} logged in", "Order {OrderId} placed" |
| `Warning` | Thứ bất thường nhưng app vẫn chạy | "Retry HTTP {Url} attempt {N}", "Slow query {Ms}ms" |
| `Error` | Operation fail, cần investigate | Exception caught, business rule violation |
| `Critical` | App-level failure, có thể restart | DB connection lost, OOM, fatal startup error |

> Production default: `Information`. Trace/Debug → bật on-demand qua dynamic config (Serilog level switch) hoặc per-namespace override.

## Log Scopes — group related logs

```csharp
public async Task ProcessOrderAsync(int orderId)
{
    using var scope = _logger.BeginScope("OrderId: {OrderId}", orderId);

    _logger.LogInformation("Validating order");
    await ValidateAsync();

    _logger.LogInformation("Charging payment");
    await ChargeAsync();

    _logger.LogInformation("Order processed");
    // Mọi log trong scope này tự có {OrderId} property
}
```

> Dùng `using var scope` cho block work — mọi log trong block tự enrich. Sink cho phép filter theo OrderId.

## Sensitive Data Redaction

```csharp
// ❌ Bad — log password/token raw
_logger.LogInformation("Login: {Email} {Password}", email, password);

// ✅ Good — không log secret. Nếu cần log object, redact field
public record LoginRequest(string Email, [property: SensitiveData] string Password);

// Custom destructurer cho Serilog
config.Destructure.ByTransforming<LoginRequest>(
    req => new { req.Email, Password = "[REDACTED]" });
```

> .NET 8+ có `Microsoft.Extensions.Compliance.Redaction` cho redaction chuẩn. Cân nhắc dùng cho app GDPR/HIPAA.

## OpenTelemetry tích hợp

```bash
dotnet add package OpenTelemetry.Exporter.Console
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
```

```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddSource("MyApp")
        .AddOtlpExporter())   // export đến Tempo/Jaeger/AppInsights
    .WithMetrics(metrics => metrics
        .AddAspNetCoreInstrumentation()
        .AddHttpClientInstrumentation()
        .AddRuntimeInstrumentation()
        .AddOtlpExporter());
```

> OpenTelemetry = future-proof, vendor-neutral. Khuyến cáo cho service production-grade. Serilog vẫn dùng song song cho structured event logs.

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| `_logger.LogInformation($"User {id}")` | String interpolation mất structure | Message template `"User {UserId}"`, userId arg |
| `Console.WriteLine` cho log | Không qua pipeline, không filter, không sink | `ILogger<T>` |
| Log password/secret/token | Security breach | Redact field, dùng `[SensitiveData]` |
| Exception chỉ log message (`ex.Message`) | Mất stack trace, inner exception | `_logger.LogError(ex, "Failed to X")` |
| Log Trace/Debug ở production mặc định | Volume ngập, perf hit | `MinimumLevel.Default = Information`; bật Debug per-namespace khi cần |
| Log lặp ở mọi layer | Duplicate data, noise | Log 1 lần ở boundary (controller, handler), hoặc rely on tracing |
| Quên `await Log.CloseAndFlushAsync()` lúc shutdown | Mất logs cuối cùng | Dùng `Host.UseSerilog()` (auto handle) |
| Sink HTTP synchronous trong hot path | Block request | Dùng async sink hoặc batched (Serilog.Sinks.Async) |
| File log không rolling/retention | Disk full | `rollingInterval: Day`, `retainedFileCountLimit: N` |

## Performance

- **Async sinks**: `Serilog.Sinks.Async` wrap sinks slow (HTTP, file) — không block request thread
- **Batch sinks**: Seq, Elastic, AppInsights tự batch — set `BatchPostingLimit` phù hợp
- **Sample**: high-volume endpoint → log subset (`Information` cho 1/100 request) qua filter
- **Avoid `LogInformation` trong tight loop**: dùng `LogTrace` hoặc bỏ hẳn

## Validate

- App dùng `ILogger<T>` + Serilog (production) — không `Console.WriteLine`
- Message template với property names PascalCase
- Correlation ID middleware đặt SỚM trong pipeline
- Log levels per-namespace override (EF Core, ASP.NET → Warning thường)
- Không log sensitive data (password, token, PII)
- Exception log có ex object: `LogError(ex, "...")`
- Production có sink ngoài Console (Seq, File rolling, AppInsights, ...)
- Đã test logs bằng cách trigger error → check sink nhận đúng

## Hand off to

- App pipeline / middleware order → `aspnet-core`
- Background jobs cần log riêng → `background-jobs`
- Health checks (logs server health) → `aspnet-health-checks`
- Distributed tracing chuyên sâu → OpenTelemetry docs
