---
name: aspnet-auth-advanced
description: "Advanced authentication/authorization trong ASP.NET Core: JWT (issue + validate, refresh tokens), OAuth2/OIDC (Google/Microsoft/external IdP), ASP.NET Core Identity (scaffolding, customize User/Claims), cookie auth cho MVC/Blazor, multi-scheme (JWT cho API + Cookie cho web), custom AuthorizationHandler/Requirements, role/policy/claim-based, anti-forgery cho SPA. Use khi setup login system, integrate Google/Microsoft login, build refresh token flow, bảo mật API + web cùng app, hoặc user nói 'JWT refresh', 'OAuth Google', 'Identity scaffold', 'authorization policy'."
compatibility: "ASP.NET Core. Identity cần EF Core hoặc custom store. OIDC providers cần app registration ở provider."
---

# Advanced Authentication & Authorization

## Trigger On

- setup JWT auth với refresh token rotation
- integrate Google/Microsoft/Facebook/Azure AD login (OAuth2/OIDC)
- scaffold ASP.NET Core Identity, customize `IdentityUser`, claims
- multi-scheme: JWT cho `/api`, Cookie cho `/admin`
- custom `AuthorizationHandler` cho business rule (e.g., "user own resource")
- roles vs claims vs policies — chọn đúng pattern
- anti-forgery cho SPA + API cookie auth
- migrate từ ASP.NET Membership / IdentityServer cũ

## Documentation

- [ASP.NET Core Authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/)
- [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity)
- [OpenIdConnect](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/social/)
- [OpenIddict](https://documentation.openiddict.com/) (modern OAuth2 server cho .NET)
- [Duende IdentityServer](https://duendesoftware.com/) (commercial, IdentityServer4 successor)

## Authentication Schemes — chọn đúng

| Scheme | Khi dùng | Storage |
|---|---|---|
| **Cookie** | Server-rendered web (MVC/Razor Pages/Blazor Server) | Cookie HttpOnly, SameSite |
| **JWT Bearer** | API cho SPA/mobile/3rd-party | Token trong `Authorization: Bearer` header |
| **OpenIdConnect** | Login qua external IdP (Google, AzureAD, Auth0) | Cookie sau khi exchange code |
| **OAuth2** | Generic OAuth (GitHub, Twitter) | Cookie |
| **API Key** | Service-to-service đơn giản | Custom scheme, header |

## JWT — issue + validate + refresh

### Setup validate

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!)),
            ClockSkew = TimeSpan.FromSeconds(30)   // tighter than default 5min
        };
    });
```

### Issue access + refresh token

```csharp
public record TokenPair(string AccessToken, string RefreshToken, DateTimeOffset AccessExpiresAt);

public class TokenService(IConfiguration config, AppDbContext db, TimeProvider time)
{
    public async Task<TokenPair> IssueAsync(User user, CancellationToken ct)
    {
        // Access token — short-lived
        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(config["Jwt:Key"]!));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        var now = time.GetUtcNow();
        var accessExpires = now.AddMinutes(15);

        var accessToken = new JwtSecurityToken(
            issuer: config["Jwt:Issuer"],
            audience: config["Jwt:Audience"],
            claims: claims,
            notBefore: now.UtcDateTime,
            expires: accessExpires.UtcDateTime,
            signingCredentials: creds);

        // Refresh token — long-lived, server-side persisted (revocable)
        var refreshToken = Convert.ToBase64String(RandomNumberGenerator.GetBytes(64));

        await db.RefreshTokens.AddAsync(new RefreshToken
        {
            Token = HashToken(refreshToken),         // hash, không lưu raw
            UserId = user.Id,
            ExpiresAt = now.AddDays(30),
            CreatedAt = now
        }, ct);
        await db.SaveChangesAsync(ct);

        return new TokenPair(
            new JwtSecurityTokenHandler().WriteToken(accessToken),
            refreshToken,
            accessExpires);
    }

    public async Task<TokenPair?> RefreshAsync(string refreshToken, CancellationToken ct)
    {
        var hashed = HashToken(refreshToken);
        var stored = await db.RefreshTokens
            .Include(r => r.User)
            .FirstOrDefaultAsync(r => r.Token == hashed && r.RevokedAt == null, ct);

        if (stored is null || stored.ExpiresAt < time.GetUtcNow())
            return null;

        // Rotation — revoke cũ, issue mới
        stored.RevokedAt = time.GetUtcNow();
        await db.SaveChangesAsync(ct);

        return await IssueAsync(stored.User, ct);
    }

    private static string HashToken(string token)
    {
        return Convert.ToHexString(SHA256.HashData(Encoding.UTF8.GetBytes(token)));
    }
}
```

> **Refresh token rotation** = revoke cũ + issue mới mỗi lần refresh. Detect reuse (token đã revoke vẫn được dùng) → revoke ALL token của user (compromised).

## OAuth2 / OIDC — Google/Microsoft login

```bash
dotnet add package Microsoft.AspNetCore.Authentication.Google
dotnet add package Microsoft.AspNetCore.Authentication.MicrosoftAccount
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
```

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = "Google";
})
.AddCookie()
.AddGoogle("Google", options =>
{
    options.ClientId = builder.Configuration["Auth:Google:ClientId"]!;
    options.ClientSecret = builder.Configuration["Auth:Google:ClientSecret"]!;
    options.SaveTokens = true;   // save access_token để gọi Google API sau
    options.Scope.Add("profile");
    options.Scope.Add("email");
})
.AddOpenIdConnect("AzureAD", options =>
{
    options.Authority = "https://login.microsoftonline.com/{tenant-id}/v2.0";
    options.ClientId = builder.Configuration["Auth:AzureAD:ClientId"]!;
    options.ClientSecret = builder.Configuration["Auth:AzureAD:ClientSecret"]!;
    options.ResponseType = "code";
    options.SaveTokens = true;
    options.GetClaimsFromUserInfoEndpoint = true;
});
```

Login endpoint:
```csharp
app.MapGet("/login/google", () =>
    Results.Challenge(new AuthenticationProperties { RedirectUri = "/" }, ["Google"]));
```

## ASP.NET Core Identity

```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI   # nếu muốn UI scaffolded
```

```csharp
public class AppUser : IdentityUser<int>
{
    public string DisplayName { get; set; } = "";
    public DateTime CreatedAt { get; set; }
}

public class AppDbContext(DbContextOptions<AppDbContext> opts)
    : IdentityDbContext<AppUser, IdentityRole<int>, int>(opts) { }

builder.Services
    .AddIdentity<AppUser, IdentityRole<int>>(options =>
    {
        options.Password.RequiredLength = 10;
        options.Password.RequireNonAlphanumeric = false;
        options.Lockout.MaxFailedAccessAttempts = 5;
        options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
        options.User.RequireUniqueEmail = true;
        options.SignIn.RequireConfirmedEmail = true;
    })
    .AddEntityFrameworkStores<AppDbContext>()
    .AddDefaultTokenProviders();
```

```bash
# Scaffold UI (Razor Pages) để customize login/register
dotnet aspnet-codegenerator identity -dc AppDbContext --files "Account.Login;Account.Register"
```

## Multi-Scheme: JWT cho API + Cookie cho Web

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
})
.AddCookie(opts => opts.LoginPath = "/Account/Login")
.AddJwtBearer("Bearer", opts =>
{
    opts.TokenValidationParameters = ...;
});

// Selector — chọn scheme dựa trên path
builder.Services.AddAuthentication()
    .AddPolicyScheme("smart", "JWT or Cookie", opts =>
    {
        opts.ForwardDefaultSelector = ctx =>
            ctx.Request.Path.StartsWithSegments("/api")
                ? "Bearer"
                : CookieAuthenticationDefaults.AuthenticationScheme;
    });
```

## Authorization — Role vs Claim vs Policy

```csharp
// 1. Role-based
[Authorize(Roles = "Admin,Manager")]
public IActionResult AdminPanel() { }

// 2. Claim-based (đơn giản)
builder.Services.AddAuthorization(opts =>
{
    opts.AddPolicy("Over18", p => p.RequireClaim("age", "18", "19", "20"));
    opts.AddPolicy("HrDept", p => p.RequireClaim("dept", "HR"));
});

[Authorize(Policy = "Over18")]
public IActionResult RestrictedContent() { }

// 3. Policy với requirements + handler (phức tạp)
public class MinimumAgeRequirement(int minAge) : IAuthorizationRequirement
{
    public int MinAge { get; } = minAge;
}

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        MinimumAgeRequirement requirement)
    {
        var dob = context.User.FindFirst("dob")?.Value;
        if (DateOnly.TryParse(dob, out var d))
        {
            var age = DateTime.Today.Year - d.Year;
            if (DateOnly.FromDateTime(DateTime.Today) < d.AddYears(age)) age--;
            if (age >= requirement.MinAge)
                context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }
}

builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
builder.Services.AddAuthorization(opts =>
    opts.AddPolicy("21OrOver", p => p.AddRequirements(new MinimumAgeRequirement(21))));
```

### Resource-based authorization (user own resource)

```csharp
public class DocumentAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Document>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        OperationAuthorizationRequirement requirement,
        Document resource)
    {
        if (requirement.Name == "Read" &&
            (resource.OwnerId == context.User.GetUserId() || resource.IsPublic))
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}

// Use trong controller
public async Task<IActionResult> Get(int id, [FromServices] IAuthorizationService authz)
{
    var doc = await db.Documents.FindAsync(id);
    if (doc is null) return NotFound();

    var authResult = await authz.AuthorizeAsync(User, doc, Operations.Read);
    if (!authResult.Succeeded) return Forbid();

    return Ok(doc);
}
```

## Anti-forgery cho SPA + Cookie auth

```csharp
builder.Services.AddAntiforgery(options =>
{
    options.HeaderName = "X-CSRF-TOKEN";
    options.Cookie.Name = "XSRF-TOKEN";
    options.Cookie.HttpOnly = false;   // SPA cần đọc token này
    options.Cookie.SameSite = SameSiteMode.Strict;
});

// Endpoint cấp token cho SPA
app.MapGet("/antiforgery/token", (IAntiforgery anti, HttpContext ctx) =>
{
    var tokens = anti.GetAndStoreTokens(ctx);
    return Results.Ok(new { token = tokens.RequestToken });
}).RequireAuthorization();
```

SPA: đọc cookie `XSRF-TOKEN`, gắn vào header `X-CSRF-TOKEN` cho mọi POST/PUT/DELETE.

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| JWT secret trong `appsettings.json` commit | Leak khi push code | User Secrets (dev), KeyVault/env (prod) |
| JWT có lifetime dài (>1h) không refresh | Compromised token dùng được lâu | Access ngắn (15min) + refresh rotation |
| Refresh token lưu raw | Leak DB → user impersonation | Hash trước khi lưu (SHA256) |
| Quên revoke refresh khi logout | Token vẫn dùng được sau logout | DB `RevokedAt` flag |
| `[Authorize]` mà không scheme cụ thể trong multi-scheme app | Default scheme có thể sai | `[Authorize(AuthenticationSchemes = "Bearer")]` |
| Role hard-coded trong code (`User.IsInRole("Admin")`) | Không flexible, khó audit | Policy/claim-based |
| Cookie không `HttpOnly` + `Secure` + `SameSite` | XSS, MITM | Set 3 flag, `SameSite=Strict` (hoặc `Lax`) |
| Long-lived persistent cookie default | Session hijack | `IsPersistent = false` cho hầu hết case |
| Thiếu rate limit endpoint login | Brute force password | `AddRateLimiter` policy cho `/login` |
| `ClockSkew` default 5min cho token short-lived | Token "valid" lâu hơn intent | `ClockSkew = TimeSpan.FromSeconds(30)` |
| Trust client claim không validate | Privilege escalation | Server-side authorization, claim chỉ informational |
| Dùng password recovery với link không expire | Account takeover | Token có expiry (15min) + 1 lần dùng |

## Validate

- JWT có short access (15min) + refresh rotation
- Refresh token hash trước khi lưu DB
- Multi-scheme rõ ràng cho API vs Web
- Cookie có `HttpOnly` + `Secure` + `SameSite`
- Authorization dùng Policy/Claim, không hard-code role
- Resource-based check (user own resource) qua `IAuthorizationService`
- Rate limit cho login/refresh endpoints
- Anti-forgery cho cookie-auth SPA POST
- Identity password policy mạnh, lockout sau N fails
- Secrets không commit code (User Secrets/KeyVault)
- Email confirmation bắt buộc nếu Identity sign-up

## Hand off to

- Pipeline middleware order, basic JWT setup → `aspnet-core`
- DB schema cho User/Role → `entity-framework-core`
- Login form Razor/Blazor → `aspnet-mvc` / `blazor`
- Logging auth events → `aspnet-logging`
- Cache permissions → `aspnet-caching`
