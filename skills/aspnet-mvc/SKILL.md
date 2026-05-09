---
name: aspnet-mvc
description: "Build ASP.NET Core MVC apps with Razor views, view models, ModelState validation, Tag Helpers, View Components, and Areas. Use when project has Views/, .cshtml files, controllers returning View()/PartialView(), or when scaffolding/refactoring server-rendered web apps. Distinct from Web API (returns View, not JSON) and Razor Pages (uses Controller, not PageModel)."
compatibility: "Requires ASP.NET Core MVC project (uses Microsoft.NET.Sdk.Web with controllers + Views/ folder)."
---

# ASP.NET Core MVC

## Trigger On

- working on `.cshtml` Razor views, `_Layout.cshtml`, partial views
- controllers returning `View()`, `PartialView()`, `RedirectToAction()` (not `Ok()`/`Json()`)
- ModelState validation, model binding, ViewModel design
- Tag Helpers, View Components, HTML Helpers
- Areas, route templates, attribute routing for MVC
- migrating from older ASP.NET MVC 5 to ASP.NET Core MVC
- choosing between MVC, Razor Pages, Blazor, or API

## Documentation

- [ASP.NET Core MVC Overview](https://learn.microsoft.com/en-us/aspnet/core/mvc/overview)
- [Razor Syntax](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/razor)
- [Tag Helpers](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro)
- [View Components](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/view-components)
- [Model Binding](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding)

## When MVC vs Razor Pages vs Blazor

| Need | Use |
|---|---|
| Server-rendered pages, multi-action controllers, complex routing | **MVC** |
| Page-focused, simple CRUD, less ceremony | **Razor Pages** |
| Interactive UI, real-time updates, .NET-only stack | **Blazor** (xem skill `blazor`) |
| Pure JSON API for SPAs/mobile | **web-api** hoặc **minimal-apis** |

> Nếu repo đã có `Views/` + Controllers → MVC. Nếu có `Pages/` + `.cshtml.cs` PageModel → Razor Pages. Đừng trộn 2 pattern trong 1 feature.

## Setup

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews()
    .AddRazorRuntimeCompilation();  // dev only — auto reload .cshtml

var app = builder.Build();

app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

## Controller Patterns

### View Model — KHÔNG dùng entity trực tiếp

```csharp
// ❌ Bad — expose entity (lazy loading, over-posting risk)
public class ProductController : Controller
{
    public IActionResult Edit(int id)
    {
        var product = _db.Products.Find(id);
        return View(product);
    }
}

// ✅ Good — view model riêng cho từng view
public record ProductEditVm(int Id, string Name, decimal Price, List<SelectListItem> Categories);

public class ProductController(AppDbContext db) : Controller
{
    public async Task<IActionResult> Edit(int id)
    {
        var product = await db.Products.FindAsync(id);
        if (product is null) return NotFound();

        var vm = new ProductEditVm(
            product.Id,
            product.Name,
            product.Price,
            await db.Categories
                .Select(c => new SelectListItem(c.Name, c.Id.ToString()))
                .ToListAsync());

        return View(vm);
    }

    [HttpPost, ValidateAntiForgeryToken]
    public async Task<IActionResult> Edit(ProductEditVm vm)
    {
        if (!ModelState.IsValid) return View(vm);

        var product = await db.Products.FindAsync(vm.Id);
        if (product is null) return NotFound();

        product.Name = vm.Name;
        product.Price = vm.Price;
        await db.SaveChangesAsync();

        TempData["Success"] = "Đã lưu";
        return RedirectToAction(nameof(Index));  // PRG pattern
    }
}
```

### POST-Redirect-GET (PRG) bắt buộc

Sau khi POST thành công → `RedirectToAction`, không `return View()`. Tránh F5 resubmit, refresh nhân đôi data.

### Validation với ModelState

```csharp
public class ProductCreateVm
{
    [Required, StringLength(200)]
    public string Name { get; set; } = "";

    [Range(0.01, 1_000_000)]
    public decimal Price { get; set; }

    [Required]
    public int CategoryId { get; set; }
}

[HttpPost, ValidateAntiForgeryToken]
public async Task<IActionResult> Create(ProductCreateVm vm)
{
    if (!ModelState.IsValid) return View(vm);

    // Custom validation sau DataAnnotations
    if (await db.Products.AnyAsync(p => p.Name == vm.Name))
    {
        ModelState.AddModelError(nameof(vm.Name), "Tên đã tồn tại");
        return View(vm);
    }

    // ...
}
```

## Razor View Patterns

### Tag Helpers > HTML Helpers

```cshtml
@* ❌ HTML Helper — verbose, kém readable *@
@Html.LabelFor(m => m.Name)
@Html.TextBoxFor(m => m.Name, new { @class = "form-control" })
@Html.ValidationMessageFor(m => m.Name)

@* ✅ Tag Helper — HTML-like, IntelliSense tốt *@
<label asp-for="Name"></label>
<input asp-for="Name" class="form-control" />
<span asp-validation-for="Name" class="text-danger"></span>
```

### Layout + Sections

```cshtml
@* _Layout.cshtml *@
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - MyApp</title>
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <main>@RenderBody()</main>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>

@* View dùng Layout *@
@{
    ViewData["Title"] = "Sản phẩm";
    Layout = "_Layout";
}

@section Scripts {
    <script src="~/js/products.js"></script>
}
```

### Partial Views

```cshtml
@* Render partial — synchronous, deprecated *@
@Html.Partial("_ProductCard", item)

@* ✅ Async partial *@
<partial name="_ProductCard" model="item" />
```

### View Components — khi cần logic

Khi 1 widget cần fetch data riêng (vd cart summary, sidebar menu), dùng View Component thay vì stuff vào ViewBag:

```csharp
public class CartSummaryViewComponent(ICartService cart) : ViewComponent
{
    public async Task<IViewComponentResult> InvokeAsync()
    {
        var summary = await cart.GetSummaryAsync(User.Identity?.Name);
        return View(summary);   // Views/Shared/Components/CartSummary/Default.cshtml
    }
}
```

```cshtml
@* Trong Layout hoặc View *@
<vc:cart-summary />
```

### Display/Editor Templates

Custom render cho 1 type tự động:

```
Views/Shared/DisplayTemplates/Money.cshtml
Views/Shared/EditorTemplates/Money.cshtml
```

```cshtml
@* DisplayTemplates/Money.cshtml *@
@model decimal
<span class="money">@Model.ToString("C")</span>

@* Sử dụng *@
@Html.DisplayFor(m => m.Price, "Money")
@* hoặc tự match by type nếu property là decimal *@
```

## Routing

### Conventional vs Attribute

```csharp
// Conventional — phù hợp app CRUD truyền thống
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Attribute — chính xác hơn cho REST/SEO URL
[Route("san-pham")]
public class ProductController : Controller
{
    [Route("")]                          // GET /san-pham
    public IActionResult Index() { }

    [Route("{id:int}")]                  // GET /san-pham/123
    public IActionResult Detail(int id) { }

    [Route("danh-muc/{slug}")]           // GET /san-pham/danh-muc/dien-thoai
    public IActionResult ByCategory(string slug) { }
}
```

### Areas — chia module

```
Areas/
  Admin/
    Controllers/
      DashboardController.cs
    Views/
      Dashboard/
        Index.cshtml
```

```csharp
[Area("Admin"), Authorize(Roles = "Admin")]
public class DashboardController : Controller { }

// Program.cs
app.MapAreaControllerRoute(
    name: "admin",
    areaName: "Admin",
    pattern: "admin/{controller=Dashboard}/{action=Index}/{id?}");
```

## TempData / ViewData / ViewBag — khi nào dùng cái nào

| | Lifetime | Type | Use case |
|---|---|---|---|
| `Model` | Per view | Strongly-typed | **DEFAULT** — passing data View ⇄ Controller |
| `ViewData["X"]` | Single request | `object` (cast cần) | Title, sidebar menu data — phụ trợ |
| `ViewBag.X` | Single request | dynamic | Tương tự ViewData, không recommend (no IntelliSense) |
| `TempData["X"]` | Across redirect (1 request next) | `object` | **PRG flash messages** ("Đã lưu", "Lỗi") |

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| Trả entity DB trực tiếp về View | Lazy load, over-posting, leak schema | Dùng View Model |
| Logic nặng trong View (`@{ ... }` block lớn) | Khó test, mix UI/logic | Move sang Controller / View Component |
| `ViewBag` overuse | No IntelliSense, runtime error | Dùng strongly-typed Model |
| `return View()` sau POST thành công | F5 resubmit | PRG: `RedirectToAction` |
| Thiếu `[ValidateAntiForgeryToken]` trên POST | CSRF vulnerability | Add attribute (đã default cho Razor Pages) |
| Fat Controller (>5 actions, >300 lines) | Khó maintain, test | Tách Service layer / Mediator |
| Sync I/O (`db.Products.ToList()` trong action async) | Thread blocking | Dùng `await ToListAsync()` |
| `[FromBody]` cho form HTML | Body là form-encoded, không JSON | Bỏ `[FromBody]` (default model binding sẽ pickup form) |

## Validate

- View Model riêng cho mỗi view (không trả entity)
- POST → ModelState check → PRG redirect
- `[ValidateAntiForgeryToken]` trên mọi POST
- Tag Helpers thay HTML Helpers
- Layout + Sections cho asset (Scripts/Styles)
- View Components cho widget có data fetching
- Async/await everywhere — không sync DB call
- Areas khi có module Admin/Customer riêng biệt

## Hand off to

- API JSON cho SPA/mobile → `web-api` hoặc `minimal-apis`
- Real-time UI (chat, notification) → `blazor` (interactive) hoặc `signalr`
- ORM concerns → `entity-framework-core`
- Auth → `aspnet-core` (JWT / cookies)
