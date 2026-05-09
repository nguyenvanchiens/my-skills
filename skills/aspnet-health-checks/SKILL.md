---
name: aspnet-health-checks
description: "Setup ASP.NET Core Health Checks: AddHealthChecks() với probes cho DB (SqlServer/Npgsql), Redis, RabbitMQ, external HTTP, custom check; tách `/healthz/live` (liveness) vs `/healthz/ready` (readiness) cho Kubernetes/Docker; AspNetCore.HealthChecks.UI dashboard; tag-based filtering. Use khi deploy lên K8s/Docker, setup load balancer probe, monitor service dependencies, hoặc user nói 'health endpoint', 'liveness/readiness', 'monitor service'."
compatibility: "Requires ASP.NET Core. UI dashboard cần `AspNetCore.HealthChecks.UI` (community)."
---

# ASP.NET Core Health Checks

## Trigger On

- deploy lên Kubernetes/Docker — cần `livenessProbe`, `readinessProbe` endpoints
- load balancer (nginx, AWS ALB, Azure App Gateway) cần health endpoint
- monitor service health (dependencies up/down, latency)
- separate liveness vs readiness checks
- thêm probe cho DB, Redis, queue, external API
- dashboard health visualization (`AspNetCore.HealthChecks.UI`)

## Documentation

- [Health Checks in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks)
- [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) — community packages cho 50+ checks

## Liveness vs Readiness — bắt buộc tách 2

| | Liveness | Readiness |
|---|---|---|
| Hỏi gì | App có còn alive không? | App có sẵn sàng nhận traffic không? |
| Fail = | Restart container | Tạm thời ngưng route traffic |
| Check gì | App process responsive (chỉ self-check, KHÔNG check dependencies) | DB, Redis, dependencies, init complete |
| Endpoint | `/healthz/live` | `/healthz/ready` |
| K8s probe | `livenessProbe` | `readinessProbe` |

> **Quan trọng**: liveness KHÔNG check DB. Nếu DB down, app không cần restart — chỉ readiness fail (k8s ngưng route traffic, app vẫn chạy). Liveness check DB → DB down → restart loop vô tận.

## Setup cơ bản

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddSqlServer(
        connectionString: builder.Configuration.GetConnectionString("Default")!,
        name: "sql",
        failureStatus: HealthStatus.Unhealthy,
        tags: ["db", "ready"])
    .AddRedis(
        redisConnectionString: builder.Configuration.GetConnectionString("Redis")!,
        name: "redis",
        tags: ["cache", "ready"])
    .AddUrlGroup(
        new Uri("https://api.partner.com/health"),
        name: "partner-api",
        tags: ["external", "ready"]);

var app = builder.Build();

// Liveness — chỉ check app self (không check tag)
app.MapHealthChecks("/healthz/live", new HealthCheckOptions
{
    Predicate = _ => false,   // không chạy bất kỳ registered check, chỉ trả OK nếu app live
    ResponseWriter = WriteHealthResponse
});

// Readiness — chạy mọi check có tag "ready"
app.MapHealthChecks("/healthz/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready"),
    ResponseWriter = WriteHealthResponse
});

app.Run();

static Task WriteHealthResponse(HttpContext context, HealthReport report)
{
    context.Response.ContentType = "application/json";
    var response = new
    {
        status = report.Status.ToString(),
        totalDuration = report.TotalDuration.TotalMilliseconds,
        checks = report.Entries.Select(e => new
        {
            name = e.Key,
            status = e.Value.Status.ToString(),
            description = e.Value.Description,
            duration = e.Value.Duration.TotalMilliseconds,
            error = e.Value.Exception?.Message
        })
    };
    return context.Response.WriteAsync(JsonSerializer.Serialize(response));
}
```

## Common probes — packages

```bash
# Database
dotnet add package AspNetCore.HealthChecks.SqlServer
dotnet add package AspNetCore.HealthChecks.Npgsql       # PostgreSQL
dotnet add package AspNetCore.HealthChecks.MySql

# Cache
dotnet add package AspNetCore.HealthChecks.Redis

# Messaging
dotnet add package AspNetCore.HealthChecks.RabbitMQ
dotnet add package AspNetCore.HealthChecks.Kafka

# External
dotnet add package AspNetCore.HealthChecks.Uris

# Hệ thống
dotnet add package AspNetCore.HealthChecks.System       # disk, memory
```

## Custom Health Check

```csharp
public class StripeHealthCheck(IStripeClient stripe) : IHealthCheck
{
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            var sw = Stopwatch.StartNew();
            await stripe.GetService<AccountService>().GetSelfAsync(cancellationToken: cancellationToken);
            sw.Stop();

            // Degraded nếu chậm
            if (sw.ElapsedMilliseconds > 2000)
                return HealthCheckResult.Degraded($"Stripe slow: {sw.ElapsedMilliseconds}ms");

            return HealthCheckResult.Healthy($"Stripe OK ({sw.ElapsedMilliseconds}ms)");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Stripe API unreachable", ex);
        }
    }
}

// Register
builder.Services.AddHealthChecks()
    .AddCheck<StripeHealthCheck>("stripe", tags: ["external", "ready"]);
```

## Health Status — 3 levels

| Status | Khi dùng | Effect |
|---|---|---|
| `Healthy` | Working bình thường | LB route traffic |
| `Degraded` | Working nhưng chậm/limited | LB route nhưng có thể alert |
| `Unhealthy` | Không serve được | LB ngưng route, k8s restart (nếu liveness) |

## Health Check UI Dashboard

```bash
dotnet add package AspNetCore.HealthChecks.UI
dotnet add package AspNetCore.HealthChecks.UI.Client
dotnet add package AspNetCore.HealthChecks.UI.InMemory.Storage   # hoặc SQLite/SqlServer
```

```csharp
builder.Services.AddHealthChecksUI(opt =>
{
    opt.SetEvaluationTimeInSeconds(30);
    opt.MaximumHistoryEntriesPerEndpoint(60);
    opt.AddHealthCheckEndpoint("My App", "/healthz/ready");
}).AddInMemoryStorage();

app.MapHealthChecks("/healthz/ready", new HealthCheckOptions
{
    Predicate = check => check.Tags.Contains("ready"),
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse   // format cho UI
});

app.MapHealthChecksUI(opt => opt.UIPath = "/healthz/ui");
```

Truy cập `/healthz/ui` → dashboard với history, charts.

## Kubernetes Probe Config

```yaml
# deployment.yaml
spec:
  containers:
  - name: myapp
    livenessProbe:
      httpGet:
        path: /healthz/live
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 30
      timeoutSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /healthz/ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 2
```

## Tag Strategy

```csharp
.AddSqlServer(..., tags: ["db", "ready", "critical"])
.AddRedis(..., tags: ["cache", "ready"])
.AddUrlGroup(..., tags: ["external", "ready", "non-critical"])

// Multiple endpoints với filter khác nhau
app.MapHealthChecks("/healthz/critical", new()
{
    Predicate = c => c.Tags.Contains("critical")
});
app.MapHealthChecks("/healthz/full", new()
{
    Predicate = _ => true
});
```

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| Liveness check DB | DB down → k8s restart loop | Liveness `Predicate = _ => false` (chỉ self) |
| Single endpoint cho cả 2 | Không tách behavior | `/healthz/live` + `/healthz/ready` riêng |
| No timeout trong custom check | Probe hang → k8s tưởng app dead | Pass `cancellationToken`, set timeout |
| Check external API trong liveness | Network blip → restart | External checks chỉ ở readiness |
| Health endpoint không protected nhưng leak detail | Info disclosure (DB version, schema) | Filter response, hoặc require auth cho `/healthz/full` |
| `Predicate = _ => true` cho liveness | Chạy mọi check (slow) | Chỉ readiness chạy full |
| Quên handle exception trong custom check | `IHealthCheck` lib auto wrap nhưng custom logic dễ throw | Try/catch return `Unhealthy(ex)` |
| Probe interval quá ngắn (1s) | Spam DB | `periodSeconds: 10-30` |

## Performance

- **Cache health result**: nếu probe heavy (full DB query), cache 5-10s tránh hammer DB
- **Parallel execution**: built-in `HealthCheckService` chạy parallel — đặt timeout reasonable cho mỗi check
- **Lazy register**: probe chỉ chạy khi endpoint hit, không background

## Validate

- Tách `/healthz/live` (no deps) và `/healthz/ready` (full deps)
- Liveness KHÔNG check DB/Redis/external
- Readiness include all critical deps với tags
- Custom check có exception handling, return `Unhealthy(ex)`
- K8s probe config với `initialDelay`, `period`, `timeout`, `failureThreshold` reasonable
- Health endpoint không leak sensitive info (DB connection string, schema)
- Timeout trong custom check honor `CancellationToken`
- (Optional) UI dashboard cho ops monitoring

## Hand off to

- Logging health events → `aspnet-logging`
- Hosting/middleware order → `aspnet-core`
- DB connection management → `entity-framework-core`
- Background task health → `background-jobs`
