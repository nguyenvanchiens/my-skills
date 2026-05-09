---
name: aspnet-caching
description: "ASP.NET Core caching patterns: `IMemoryCache` (in-process), `IDistributedCache` (Redis/SQL Server), `HybridCache` (.NET 9 unified, L1+L2), Output Caching (.NET 7+), Response Caching, ResponseCache attribute. Cache strategies (cache-aside, write-through, refresh-ahead), key conventions, TTL/sliding expiration, invalidation, stampede prevention. Use khi user nói 'app chậm khi load list', 'DB query lặp', 'cache với Redis', 'output caching', 'session state', hoặc khi setup caching layer cho production."
compatibility: "Requires ASP.NET Core. `HybridCache` cần .NET 9+. Output Caching cần .NET 7+. Redis cần Redis server."
---

# ASP.NET Core Caching

## Trigger On

- DB query trùng lặp giữa requests, app chậm khi load data ít thay đổi
- thêm Redis vào solution (distributed scenario, multi-instance)
- output caching cho HTML response, response caching cho API
- session state external (Redis backing)
- chọn giữa `IMemoryCache`, `IDistributedCache`, `HybridCache` (.NET 9)
- cache invalidation, TTL strategies, stampede prevention
- setup `Microsoft.Extensions.Caching.StackExchangeRedis` hoặc `Microsoft.Extensions.Caching.SqlServer`

## Documentation

- [Caching in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/overview)
- [HybridCache .NET 9](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/hybrid)
- [Output caching](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/output)
- [StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/)

## Quick Decision

```
1 instance app, data fits memory       → IMemoryCache
Multi-instance app (load balanced)     → IDistributedCache (Redis)
.NET 9+, muốn L1 (mem) + L2 (Redis)   → HybridCache (recommended cho new app)
Cache HTTP response (full page/API)    → Output Caching (.NET 7+) hoặc Response Caching
Cache HTML render output Razor/Blazor  → Output Caching
Session state (user-specific)          → Distributed session (Redis-backed)
```

## `IMemoryCache` — single instance, fastest

```csharp
// Program.cs
builder.Services.AddMemoryCache(options =>
{
    options.SizeLimit = 1024;          // tổng size units (entries phải set Size)
    options.CompactionPercentage = 0.25;
});

// Usage
public class ProductService(IMemoryCache cache, AppDbContext db)
{
    public async Task<Product?> GetAsync(int id, CancellationToken ct)
    {
        var key = $"product:{id}";

        if (cache.TryGetValue<Product>(key, out var cached))
            return cached;

        var product = await db.Products.FindAsync([id], ct);
        if (product is null) return null;

        cache.Set(key, product, new MemoryCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10),
            SlidingExpiration = TimeSpan.FromMinutes(2),
            Size = 1,
            Priority = CacheItemPriority.Normal
        });

        return product;
    }
}
```

> ⚠️ Mất khi app restart, không share giữa instance. Chỉ dùng khi: 1 instance, hoặc data tolerable mất sau restart.

### `GetOrCreateAsync` — pattern cache-aside ngắn gọn

```csharp
public async Task<Product?> GetAsync(int id, CancellationToken ct)
{
    return await cache.GetOrCreateAsync($"product:{id}", async entry =>
    {
        entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10);
        return await db.Products.FindAsync([id], ct);
    });
}
```

> ⚠️ `GetOrCreateAsync` KHÔNG có stampede protection — nhiều request concurrent sẽ chạy factory nhiều lần. Dùng `HybridCache` hoặc lock thủ công nếu cần.

## `IDistributedCache` — multi-instance / Redis

```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

```csharp
// Program.cs
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis");
    options.InstanceName = "MyApp:";   // prefix cho mọi key
});

// Usage
public class ProductService(IDistributedCache cache, AppDbContext db)
{
    public async Task<Product?> GetAsync(int id, CancellationToken ct)
    {
        var key = $"product:{id}";

        var cached = await cache.GetStringAsync(key, ct);
        if (cached is not null)
            return JsonSerializer.Deserialize<Product>(cached);

        var product = await db.Products.FindAsync([id], ct);
        if (product is null) return null;

        await cache.SetStringAsync(key,
            JsonSerializer.Serialize(product),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
            }, ct);

        return product;
    }
}
```

> Redis = distributed cache phổ biến nhất. Alternative: SQL Server (`Microsoft.Extensions.Caching.SqlServer`) cho team không có Redis infrastructure.

## `HybridCache` — .NET 9+ recommended

`HybridCache` là API mới (.NET 9, in-progress preview cho .NET 8) gộp L1 (memory) + L2 (distributed). Tự handle stampede, serialization, tagging.

```bash
dotnet add package Microsoft.Extensions.Caching.Hybrid
```

```csharp
builder.Services.AddHybridCache(options =>
{
    options.DefaultEntryOptions = new HybridCacheEntryOptions
    {
        Expiration = TimeSpan.FromMinutes(10),
        LocalCacheExpiration = TimeSpan.FromMinutes(2)
    };
});
builder.Services.AddStackExchangeRedisCache(...);   // L2 backend

// Usage
public class ProductService(HybridCache cache, AppDbContext db)
{
    public async Task<Product?> GetAsync(int id, CancellationToken ct)
    {
        return await cache.GetOrCreateAsync(
            $"product:{id}",
            async (token) => await db.Products.FindAsync([id], token),
            tags: ["product", $"product:{id}"],
            cancellationToken: ct);
    }

    // Invalidate by tag
    public async Task InvalidateAsync(int id, CancellationToken ct)
    {
        await cache.RemoveByTagAsync($"product:{id}", ct);
    }
}
```

**Pros**: stampede protection out-of-box, tag invalidation, type-safe (no manual JSON), 2-tier built-in.
**Cons**: .NET 9 stable (preview NuGet cho .NET 8).

## Output Caching — .NET 7+

Cache full HTTP response (HTML / JSON / file) trên server, request tiếp theo skip endpoint logic.

```csharp
// Program.cs
builder.Services.AddOutputCache(options =>
{
    options.AddBasePolicy(b => b.Expire(TimeSpan.FromMinutes(5)));

    options.AddPolicy("ProductDetail", b => b
        .Expire(TimeSpan.FromHours(1))
        .SetVaryByQuery("id")
        .Tag("products"));
});

var app = builder.Build();
app.UseOutputCache();

// Endpoint với policy
app.MapGet("/products/{id}", async (int id, AppDbContext db) =>
{
    var product = await db.Products.FindAsync(id);
    return Results.Ok(product);
}).CacheOutput("ProductDetail");

// Hoặc trên controller
[OutputCache(PolicyName = "ProductDetail")]
public IActionResult Detail(int id) { }

// Invalidate by tag
app.MapPost("/admin/cache/products/clear", async (IOutputCacheStore store) =>
{
    await store.EvictByTagAsync("products", default);
    return Results.NoContent();
});
```

> Khác Response Caching (header-based, client/proxy cache): **Output Caching = server-side cache**, không phụ thuộc client header. Recommend cho hầu hết case.

## Response Caching (HTTP cache headers)

```csharp
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
public IActionResult GetPublicData() { }
```

Set `Cache-Control` header → browser/CDN/proxy cache. Không lưu trên server.

> Use case: static-ish public API. Combine với CDN (CloudFront, Cloudflare) cho hiệu quả.

## Cache-aside Pattern (most common)

```
1. Check cache by key
2. Hit → return
3. Miss → load from DB → set cache → return
```

Đã shown ở các example trên. Đây là pattern chuẩn cho 90% case.

## Stampede Prevention (cache miss + concurrent requests)

Vấn đề: cache expire → 1000 request đồng loạt miss → 1000 DB query.

**Solutions**:

1. **`HybridCache`** — built-in stampede protection (recommended)
2. **Manual lock** trong factory (in-process):
```csharp
private static readonly SemaphoreSlim _lock = new(1, 1);

public async Task<Product> GetAsync(int id)
{
    if (cache.TryGetValue<Product>(key, out var v)) return v;

    await _lock.WaitAsync();
    try
    {
        if (cache.TryGetValue<Product>(key, out v)) return v;   // double-check
        v = await db.Products.FindAsync(id);
        cache.Set(key, v, TimeSpan.FromMinutes(10));
        return v;
    }
    finally { _lock.Release(); }
}
```
3. **Refresh-ahead**: background job refresh cache trước khi expire

## Cache Key Conventions

```
<entity-type>:<id>                    → product:123
<entity-type>:list:<filter-hash>      → product:list:abc123
<user-scoped>:<user-id>:<resource>    → user:42:cart
<feature>:<version>:<key>             → search:v2:popular-keywords
```

> Prefix `<app-name>:` (set qua `InstanceName`) để share Redis với apps khác mà không collide.

## TTL Strategies

| Pattern | TTL |
|---|---|
| Reference data (categories, configs) | 1h - 1 day, sliding |
| User session data | 30 min sliding |
| Search results / list pages | 5-15 min |
| Detail pages | 10-30 min |
| User-specific data | 5-10 min |
| Highly dynamic (real-time) | KHÔNG cache, hoặc <30s |

## Invalidation Patterns

```csharp
// 1. Time-based (TTL) — đơn giản nhất, eventual consistency
cache.Set(key, value, TimeSpan.FromMinutes(10));

// 2. Event-based — khi update DB, evict cache
public async Task UpdateAsync(Product p)
{
    db.Update(p);
    await db.SaveChangesAsync();
    await cache.RemoveAsync($"product:{p.Id}");
    await cache.RemoveAsync("product:list");          // list cache stale
}

// 3. Tag-based (HybridCache + Output Cache) — invalidate nhóm key
await cache.RemoveByTagAsync("products");
```

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| Cache mọi thứ "để cho nhanh" | Memory bloat, stale data, debug khó | Cache theo measurement — profile xác định hot path trước |
| Cache user-specific data trong shared `IMemoryCache` không có user prefix | Cross-user data leak | Key có user ID |
| Quên invalidate khi update | Stale data | Event-based invalidation hoặc TTL ngắn |
| TTL quá dài (`days`) | Stale data lâu | TTL phù hợp tốc độ thay đổi data |
| `IMemoryCache` cho multi-instance app | Mỗi instance cache riêng → inconsistent | `IDistributedCache`/Redis |
| Dùng `IDistributedCache` cho hot-path 1ms | Network latency Redis 1-5ms — slower | `HybridCache` (L1 mem) hoặc `IMemoryCache` |
| Cache `null` mà code không handle | NullRefException khi data thực sự không tồn tại | Cache marker "NotFound" hoặc skip cache |
| Stampede do hot key expire | DB spike khi cache miss | `HybridCache` hoặc lock |
| Không set `SizeLimit` cho `IMemoryCache` | Memory unbounded | `SizeLimit` + `Size` per entry |
| Serialize lớn vào cache (>1MB/entry) | Network/memory pressure | Cache reference/ID, fetch detail riêng |
| Cache method có CancellationToken nhưng không respect | Hang request khi client disconnect | Pass `ct` đến factory delegate |

## Performance

- **Profile trước cache**: dùng MiniProfiler/AppInsights confirm chỗ chậm là DB query, không phải gì khác
- **Measure hit rate**: log/metrics hit vs miss — <70% hit rate thì cache không đáng
- **Compress large values**: `IDistributedCache` set `BinaryFormatter` hoặc gzip JSON cho entry >50KB
- **Pipeline Redis ops**: dùng `IBatch` từ StackExchange.Redis cho multi-key fetch

## Validate

- Chọn đúng cache type (`IMemoryCache` / `IDistributedCache` / `HybridCache` / Output)
- Key có namespace prefix, không collision
- TTL phù hợp tốc độ thay đổi data
- Invalidation handle khi update (event-based hoặc tag-based)
- Stampede protection cho hot key (`HybridCache` hoặc lock)
- Multi-instance app dùng distributed cache, không `IMemoryCache`
- Không cache sensitive data trừ khi đã encrypt
- Memory cache có `SizeLimit`
- Đã measure hit rate trên production

## Hand off to

- App pipeline order, middleware → `aspnet-core`
- Cache HTTP response → Output Caching section trên (đã cover)
- Database tier optimization → `optimizing-ef-core-queries`
- Background refresh job → `background-jobs`
