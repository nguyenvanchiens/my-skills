---
name: dotnet-testing-patterns
description: ".NET test design patterns: AAA structure, mocking với Moq/NSubstitute, FluentAssertions, integration test với `WebApplicationFactory<T>` (in-memory ASP.NET Core test server), test isolation với Testcontainers (real DB/Redis/RabbitMQ), test data builders, snapshot testing, Bogus fake data, time abstraction (`TimeProvider`/`FakeTimeProvider`). Use khi viết test mới, refactor test khó maintain, integration test API, test EF Core với DB thật, hoặc user nói 'mock service', 'test API endpoint', 'test với DB', 'fake data'."
compatibility: "Bổ sung cho skill `xunit` (chỉ cover runner). Áp dụng được với xUnit/NUnit/MSTest. Một số package (Testcontainers) cần Docker."
---

# .NET Test Design Patterns

## Trigger On

- viết unit test mới với mock dependencies
- integration test API endpoint với `WebApplicationFactory`
- test EF Core code với DB thật (không InMemory provider)
- test với DateTime/Random — cần inject `TimeProvider`
- test data setup repetitive — refactor sang Builder pattern
- assertion verbose — refactor sang FluentAssertions
- chọn giữa Moq, NSubstitute, FakeItEasy, Microsoft.Extensions.Hosting.Testing

> Skill này focus DESIGN. Skill `xunit` cover RUNNER (v2/v3, VSTest/MTP). Dùng song song.

## Documentation

- [xUnit](https://xunit.net/)
- [Moq](https://github.com/devlooped/moq)
- [NSubstitute](https://nsubstitute.github.io/)
- [FluentAssertions](https://fluentassertions.com/)
- [Testcontainers .NET](https://dotnet.testcontainers.org/)
- [WebApplicationFactory](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)
- [Bogus](https://github.com/bchavez/Bogus)

## AAA Pattern — chuẩn cho mọi test

```csharp
[Fact]
public async Task Place_ValidOrder_ReturnsOrderId()
{
    // Arrange
    var customer = new Customer { Id = 1, Email = "a@b.com" };
    var dto = new PlaceOrderDto { CustomerId = 1, Items = [new(1, 2)] };
    var sut = new OrderService(_db, _emailService);

    // Act
    var result = await sut.PlaceAsync(dto);

    // Assert
    result.Should().NotBeNull();
    result.OrderId.Should().BeGreaterThan(0);
    await _emailService.Received(1).SendOrderConfirmationAsync(customer.Email);
}
```

**Rule**:
- 1 logical assertion per test (multiple `.Should()` OK nếu cùng concept)
- Test name: `Method_Scenario_ExpectedBehavior` hoặc `Should_X_When_Y`
- Không if/else trong test (split thành 2 tests)
- Không sleep/delay (use `TimeProvider`/test scheduler)

## Mocking — chọn lib

| Lib | Pros | Cons |
|---|---|---|
| **Moq** | Phổ biến nhất, ecosystem lớn | Recently controversial (telemetry), syntax verbose |
| **NSubstitute** | Syntax sạch, lambda thân thiện | Ít powerful cho edge case |
| **FakeItEasy** | Syntax tự nhiên | Documentation kém |

> Recommend **NSubstitute** cho project mới (clean syntax). Existing project dùng Moq → giữ nguyên.

### NSubstitute

```csharp
var emailService = Substitute.For<IEmailService>();

emailService
    .SendAsync(Arg.Any<string>(), Arg.Any<string>(), Arg.Any<string>())
    .Returns(Task.CompletedTask);

// Act ...

await emailService.Received(1).SendAsync("user@x.com", Arg.Any<string>(), Arg.Any<string>());
await emailService.DidNotReceive().SendAsync("admin@x.com", Arg.Any<string>(), Arg.Any<string>());
```

### Moq

```csharp
var emailService = new Mock<IEmailService>();

emailService
    .Setup(s => s.SendAsync(It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()))
    .Returns(Task.CompletedTask);

// Act ...

emailService.Verify(s => s.SendAsync("user@x.com", It.IsAny<string>(), It.IsAny<string>()), Times.Once);
```

## FluentAssertions — readable assertion

```bash
dotnet add package FluentAssertions
```

```csharp
// ❌ Built-in
Assert.NotNull(result);
Assert.Equal(2, result.Count);
Assert.True(result.All(x => x.IsActive));
Assert.Contains(result, x => x.Id == 1);

// ✅ FluentAssertions
result.Should().NotBeNull();
result.Should().HaveCount(2);
result.Should().OnlyContain(x => x.IsActive);
result.Should().ContainSingle(x => x.Id == 1);

// Object equality
order.Should().BeEquivalentTo(expectedOrder, opts => opts
    .Excluding(o => o.CreatedAt)
    .ComparingByMembers<Money>());

// Exception
await sut.Invoking(s => s.PlaceAsync(invalidDto))
    .Should().ThrowAsync<ValidationException>()
    .WithMessage("*Email is required*");
```

## Test Data Builder

Khi setup data lặp lại, refactor sang builder:

```csharp
public class OrderBuilder
{
    private int _id = 1;
    private int _customerId = 1;
    private decimal _total = 100m;
    private OrderStatus _status = OrderStatus.Pending;
    private List<OrderItem> _items = [];

    public OrderBuilder WithId(int id) { _id = id; return this; }
    public OrderBuilder ForCustomer(int customerId) { _customerId = customerId; return this; }
    public OrderBuilder Paid() { _status = OrderStatus.Paid; return this; }
    public OrderBuilder Cancelled() { _status = OrderStatus.Cancelled; return this; }
    public OrderBuilder WithItems(params OrderItem[] items) { _items.AddRange(items); return this; }

    public Order Build() => new()
    {
        Id = _id,
        CustomerId = _customerId,
        Total = _total,
        Status = _status,
        Items = _items
    };

    public static implicit operator Order(OrderBuilder b) => b.Build();
}

// Usage
var order = new OrderBuilder()
    .ForCustomer(42)
    .Paid()
    .WithItems(new(1, 2), new(2, 1))
    .Build();
```

## Bogus — fake data realistic

```bash
dotnet add package Bogus
```

```csharp
var faker = new Faker<Customer>()
    .RuleFor(c => c.Name, f => f.Name.FullName())
    .RuleFor(c => c.Email, (f, c) => f.Internet.Email(c.Name))
    .RuleFor(c => c.Phone, f => f.Phone.PhoneNumber())
    .RuleFor(c => c.Address, f => f.Address.FullAddress());

var single = faker.Generate();
var batch = faker.Generate(100);
```

> Hữu ích cho seed test DB, generate lots of records, demo data.

## Time Abstraction — `TimeProvider` (.NET 8+)

```csharp
public class OrderService(AppDbContext db, TimeProvider time)
{
    public async Task<Order> PlaceAsync(PlaceOrderDto dto)
    {
        var order = new Order
        {
            CustomerId = dto.CustomerId,
            CreatedAt = time.GetUtcNow(),                    // không DateTime.UtcNow
            ExpiresAt = time.GetUtcNow().AddHours(24)
        };
        db.Orders.Add(order);
        await db.SaveChangesAsync();
        return order;
    }
}
```

```csharp
// Test với FakeTimeProvider
[Fact]
public async Task Place_SetsExpiryAt24Hours()
{
    var time = new FakeTimeProvider(DateTimeOffset.Parse("2026-01-15T10:00:00Z"));
    var sut = new OrderService(_db, time);

    var order = await sut.PlaceAsync(_dto);

    order.CreatedAt.Should().Be(time.GetUtcNow());
    order.ExpiresAt.Should().Be(time.GetUtcNow().AddHours(24));
}
```

```bash
dotnet add package Microsoft.Extensions.TimeProvider.Testing
```

## Integration Test — `WebApplicationFactory<T>`

Test toàn pipeline (middleware, routing, DI, controllers) với in-memory test server.

```csharp
public class OrderApiTests : IClassFixture<WebApplicationFactory<Program>>, IAsyncLifetime
{
    private readonly WebApplicationFactory<Program> _factory;
    private HttpClient _client = null!;

    public OrderApiTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.UseEnvironment("Testing");
            builder.ConfigureServices(services =>
            {
                // Override real services với fake/mock cho test
                services.RemoveAll<IEmailSender>();
                services.AddSingleton<IEmailSender, FakeEmailSender>();

                // Replace DB với in-memory hoặc Testcontainers
                services.RemoveAll<DbContextOptions<AppDbContext>>();
                services.AddDbContext<AppDbContext>(opts =>
                    opts.UseSqlite("Data Source=:memory:"));
            });
        });
    }

    public Task InitializeAsync()
    {
        _client = _factory.CreateClient();
        return Task.CompletedTask;
    }

    public Task DisposeAsync() => Task.CompletedTask;

    [Fact]
    public async Task Post_PlaceOrder_Returns201()
    {
        // Arrange
        var dto = new PlaceOrderDto { CustomerId = 1, Items = [new(1, 2)] };

        // Act
        var response = await _client.PostAsJsonAsync("/api/orders", dto);

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        var order = await response.Content.ReadFromJsonAsync<OrderDto>();
        order!.Id.Should().BeGreaterThan(0);
    }

    [Fact]
    public async Task Get_OrderWithoutAuth_Returns401()
    {
        var response = await _client.GetAsync("/api/orders/1");
        response.StatusCode.Should().Be(HttpStatusCode.Unauthorized);
    }
}
```

```csharp
// Program.cs cần partial cho tests reference
public partial class Program { }
```

## Testcontainers — DB thật trong test

EF Core InMemory provider có nhiều khác biệt với SQL Server thật (transaction, raw SQL, migration). Production-grade test → dùng container.

```bash
dotnet add package Testcontainers.MsSql       # hoặc PostgreSql, MySql, Redis
```

```csharp
public class IntegrationTestFixture : IAsyncLifetime
{
    public MsSqlContainer DbContainer { get; } = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .WithPassword("Strong!Pass1")
        .Build();

    public string ConnectionString => DbContainer.GetConnectionString();

    public async Task InitializeAsync()
    {
        await DbContainer.StartAsync();

        // Apply migrations
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseSqlServer(ConnectionString)
            .Options;
        await using var db = new AppDbContext(options);
        await db.Database.MigrateAsync();
    }

    public Task DisposeAsync() => DbContainer.DisposeAsync().AsTask();
}

[Collection("Integration")]
public class OrderRepositoryTests(IntegrationTestFixture fixture) : IAsyncLifetime
{
    public async Task InitializeAsync()
    {
        // Reset DB state per test class
        await using var db = new AppDbContext(...);
        await db.Database.ExecuteSqlRawAsync("DELETE FROM Orders");
    }

    public Task DisposeAsync() => Task.CompletedTask;

    [Fact]
    public async Task Save_NewOrder_Persists() { /* ... */ }
}
```

> Trade-off: chậm hơn (~3-10s startup container) vs realistic (catch SQL-specific bugs). Recommend cho integration test layer DB.

## Snapshot Testing (cho output phức tạp)

```bash
dotnet add package Verify.Xunit
```

```csharp
[Fact]
public async Task GenerateInvoice_MatchesSnapshot()
{
    var invoice = await sut.GenerateAsync(orderId);
    await Verify(invoice);
}
```

Lần đầu tạo file `Test.GenerateInvoice_MatchesSnapshot.verified.txt`. Lần sau diff với file đó. Hữu ích cho HTML/JSON/PDF output.

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| `DateTime.UtcNow` trong code SUT | Test flaky theo giờ chạy | `TimeProvider` injection |
| `Task.Delay`/`Thread.Sleep` trong test | Slow + flaky | Mock async, hoặc `FakeTimeProvider.Advance` |
| Test depends on order chạy | Brittle, hard parallelize | Mỗi test independent, reset state |
| Shared mutable state giữa tests | Race condition khi parallel | Per-test fixture, fresh data |
| Mock DbContext (giả lập IQueryable) | EF translate khác in-memory | Testcontainers hoặc SQLite InMemory |
| 1 test assert nhiều unrelated thing | Khó debug khi fail | Split tests, 1 concept/test |
| Magic strings / numbers trong assert | Maintenance nightmare | Constants hoặc data-driven |
| `Assert.Equal(true, ...)` | Verbose | `result.Should().BeTrue()` |
| In-memory EF provider cho DbContext test | Behavior khác SQL Server (transactions, FK, raw SQL) | Testcontainers với SQL Server thật |
| Test không reset DB state | Cross-contamination | Per-class container hoặc `Respawn` lib |
| Mock concrete class | Mock không track behavior chuẩn | Mock interface |
| Comment ra disabled test (`// [Fact]`) | Lose visibility | `[Fact(Skip = "reason: ...")]` |

## Test Naming Conventions

```csharp
// Pattern 1: Method_Scenario_Expected
PlaceOrder_NoItems_ThrowsValidationException
GetUser_NotFound_Returns404

// Pattern 2: Should_X_When_Y
Should_ReturnOrderId_When_PlaceValidOrder
Should_Return404_When_UserNotFound

// Pattern 3: Given/When/Then (BDD)
GivenInvalidEmail_WhenRegister_ThenReturnsValidationError
```

> Pick 1 pattern và stick với nó toàn project.

## Validate

- AAA structure rõ ràng (Arrange/Act/Assert)
- 1 concept assertion per test
- Test name có scenario + expected
- Inject `TimeProvider`, không `DateTime.UtcNow` trong SUT
- Mock interface, không concrete class
- FluentAssertions cho readable assert
- Builder pattern cho data setup phức tạp
- `WebApplicationFactory<Program>` cho integration test API
- Testcontainers cho test EF Core với DB thật (production-grade)
- Test isolation — không depend order, reset state per test
- Skip test có reason rõ ràng (`Skip = "..."`), không comment-out

## Hand off to

- Runner config (xUnit v2/v3, VSTest/MTP) → `xunit`
- Mocking patterns chi tiết per lib → official docs
- Performance test → BenchmarkDotNet (chuyên skill khác)
- API integration trong real env → `aspnet-core`
