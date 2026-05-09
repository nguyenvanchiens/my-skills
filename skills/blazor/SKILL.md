---
name: blazor
description: "Build Blazor apps on .NET 8+: unified Blazor Web App với render modes (SSR, Stream, Server, WebAssembly, Auto), components/parameters/EventCallback, lifecycle, EditForm validation, JS interop, state management. Use when project has .razor files, Components/ folder, App.razor, MainLayout.razor, or _Imports.razor."
compatibility: "Requires Blazor project (Microsoft.NET.Sdk.Web với @rendermode hoặc rfp Components/App.razor). Recommended .NET 8+ vì unified Blazor Web App model."
---

# Blazor

## Trigger On

- working on `.razor` files, `.razor.cs` code-behind, `_Imports.razor`
- choosing between Server, WebAssembly, Auto, hoặc SSR render mode
- component parameters, `EventCallback`, cascading values, two-way binding `@bind`
- component lifecycle (`OnInitializedAsync`, `OnAfterRenderAsync`, `ShouldRender`)
- `EditForm` + DataAnnotations / FluentValidation
- JS interop (`IJSRuntime.InvokeAsync`)
- state management (component, service-based, Fluxor/Blazored)
- migrate Blazor Server-only / WASM-only sang unified Blazor Web App (.NET 8)

## Documentation

- [Blazor Overview](https://learn.microsoft.com/en-us/aspnet/core/blazor/)
- [Render Modes](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/render-modes)
- [Components](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/)
- [Forms](https://learn.microsoft.com/en-us/aspnet/core/blazor/forms/)
- [JS Interop](https://learn.microsoft.com/en-us/aspnet/core/blazor/javascript-interoperability/)

## Render Modes (.NET 8+ unified)

Đây là điểm quan trọng nhất khi làm Blazor hiện đại — pick đúng render mode theo context của TỪNG component, không pick globally.

| Mode | Code chạy ở | First render | Interactive | Use case |
|---|---|---|---|---|
| **Static SSR** (default) | Server | Server HTML | ❌ Không (chỉ HTML) | Marketing page, blog, content tĩnh |
| **Stream Rendering** | Server | Server (streaming) | ❌ | Page có data slow load — render skeleton + stream khi data ready |
| **InteractiveServer** | Server | Server | ✓ qua SignalR | Admin dashboard, internal tool, complex form |
| **InteractiveWebAssembly** | Browser | Server (prerender) → Browser | ✓ client-side | App offline-capable, low server load |
| **InteractiveAuto** | Server (initial) → WASM | Server | ✓ chuyển dần | Best UX nhưng require WASM download lần đầu |

```razor
@* Per-component render mode *@
@page "/dashboard"
@rendermode InteractiveServer

<MyDashboard />

@* Per-component khi gọi *@
<Counter @rendermode="InteractiveWebAssembly" />
```

### Quyết nhanh

- App marketing/blog → `Static SSR` (default), không thêm gì
- Admin/internal có form, table interactive → `InteractiveServer` (rẻ, latency phụ thuộc network)
- App khách hàng cần offline / scale lớn → `InteractiveWebAssembly` (download initial)
- Dùng feature React-like (LocalStorage, IndexedDB) → WASM
- Không chắc → `InteractiveAuto` cho UX tốt nhất

## Component Patterns

### Parameters

```razor
@* Counter.razor *@
<button @onclick="IncrementCount">@Title: @currentCount</button>

@code {
    [Parameter] public string Title { get; set; } = "Count";
    [Parameter] public int Initial { get; set; } = 0;
    [Parameter] public EventCallback<int> OnChanged { get; set; }
    [Parameter, EditorRequired] public string Required { get; set; } = "";

    private int currentCount;

    protected override void OnInitialized() => currentCount = Initial;

    private async Task IncrementCount()
    {
        currentCount++;
        await OnChanged.InvokeAsync(currentCount);   // notify parent
    }
}
```

### Two-way binding `@bind`

```razor
@* Parent *@
<MyInput @bind-Value="userName" @bind-Value:after="OnNameChanged" />

@* MyInput.razor *@
<input value="@Value" @oninput="HandleInput" />

@code {
    [Parameter] public string Value { get; set; } = "";
    [Parameter] public EventCallback<string> ValueChanged { get; set; }

    private async Task HandleInput(ChangeEventArgs e)
    {
        var newValue = e.Value?.ToString() ?? "";
        await ValueChanged.InvokeAsync(newValue);
    }
}
```

### Cascading Values — share data qua component tree

```razor
@* Parent *@
<CascadingValue Value="@currentTheme" Name="Theme">
    <Layout>
        @Body
    </Layout>
</CascadingValue>

@* Child sâu trong tree *@
@code {
    [CascadingParameter(Name = "Theme")]
    public Theme? Theme { get; set; }
}
```

> Dùng cho dữ liệu shared toàn UI (theme, current user, culture). KHÔNG abuse — dữ liệu nghiệp vụ → service DI.

### Code-behind pattern

Component lớn → tách `.razor` (markup) khỏi `.razor.cs` (logic):

```razor
@* Counter.razor *@
@inherits CounterBase

<button @onclick="Increment">Count: @Count</button>
```

```csharp
// Counter.razor.cs
public partial class CounterBase : ComponentBase
{
    [Inject] protected IMyService Service { get; set; } = null!;

    protected int Count { get; set; }

    protected async Task Increment()
    {
        Count++;
        await Service.LogAsync(Count);
    }
}
```

## Lifecycle

```csharp
protected override void OnInitialized() { /* sync init */ }
protected override async Task OnInitializedAsync() { /* async data fetch */ }
protected override void OnParametersSet() { /* khi parameter đổi */ }
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        // JS interop — chỉ chạy 1 lần
        await JS.InvokeVoidAsync("initChart", chartId);
    }
}
protected override bool ShouldRender() => /* skip render nếu return false */;
public void Dispose() { /* cleanup */ }
```

### Khi dùng cái nào

| Need | Hook |
|---|---|
| Fetch data ban đầu | `OnInitializedAsync` |
| React khi parent đổi parameter | `OnParametersSet(Async)` |
| JS interop, focus input, init 3rd-party lib | `OnAfterRenderAsync(firstRender)` |
| Tối ưu render (skip nếu state không đổi) | `ShouldRender` |
| Cleanup subscription, timer | `IDisposable.Dispose` / `IAsyncDisposable.DisposeAsync` |

## Forms — EditForm + Validation

```razor
@page "/register"

<EditForm Model="@model" OnValidSubmit="HandleSubmit" FormName="Register">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText @bind-Value="model.Email" />
    <ValidationMessage For="@(() => model.Email)" />

    <InputText @bind-Value="model.Password" type="password" />
    <ValidationMessage For="@(() => model.Password)" />

    <button type="submit">Register</button>
</EditForm>

@code {
    [SupplyParameterFromForm]
    public RegisterModel Model { get; set; } = new();

    private RegisterModel model => Model;

    public class RegisterModel
    {
        [Required, EmailAddress] public string Email { get; set; } = "";
        [Required, MinLength(8)] public string Password { get; set; } = "";
    }

    private async Task HandleSubmit()
    {
        // Submit logic
    }
}
```

> `[SupplyParameterFromForm]` mới có .NET 8+ cho SSR forms. Server-rendered forms hoạt động không cần InteractiveServer.

## State Management

| Approach | Use case |
|---|---|
| **Component state** (`@code` field) | Local UI state — counter, expand/collapse |
| **Cascading parameter** | Theme, current user, culture |
| **Scoped service** (DI) | Cross-component state trong 1 user session (Server) hoặc tab (WASM) |
| **`PersistingComponentStateSubscription`** | Persist state qua prerender → interactive transition |
| **LocalStorage** (`Blazored.LocalStorage`) | Persist qua page reload (WASM) |
| **Fluxor** / **Redux pattern** | App phức tạp, state global rõ ràng |

### Service-based state (recommended cho hầu hết case)

```csharp
public class CartState
{
    public List<CartItem> Items { get; private set; } = new();
    public event Action? OnChange;

    public void Add(CartItem item)
    {
        Items.Add(item);
        OnChange?.Invoke();
    }
}

// Program.cs
builder.Services.AddScoped<CartState>();   // Scoped = per circuit (Server) / per tab (WASM)
```

```razor
@inject CartState Cart
@implements IDisposable

<p>Cart count: @Cart.Items.Count</p>

@code {
    protected override void OnInitialized() => Cart.OnChange += StateHasChanged;
    public void Dispose() => Cart.OnChange -= StateHasChanged;
}
```

## JS Interop

```razor
@inject IJSRuntime JS

<input @ref="inputRef" />

@code {
    private ElementReference inputRef;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await inputRef.FocusAsync();
            await JS.InvokeVoidAsync("myJs.initChart", "#chart", chartData);
            var result = await JS.InvokeAsync<string>("prompt", "Enter name:");
        }
    }
}
```

```js
// wwwroot/js/site.js
window.myJs = {
    initChart: (selector, data) => { /* ... */ }
};
```

```html
@* App.razor / _Host.cshtml *@
<script src="js/site.js"></script>
```

## Authentication

```razor
@* AuthorizeView component *@
<AuthorizeView>
    <Authorized>
        <p>Hello @context.User.Identity?.Name</p>
    </Authorized>
    <NotAuthorized>
        <a href="/login">Login</a>
    </NotAuthorized>
</AuthorizeView>

@* Page-level *@
@page "/admin"
@attribute [Authorize(Roles = "Admin")]
```

## Anti-patterns

| Anti-pattern | Vấn đề | Fix |
|---|---|---|
| Đặt mọi component `@rendermode InteractiveServer` | Lãng phí SignalR connection cho component tĩnh | Per-component render mode |
| Fetch data trong `OnAfterRenderAsync` | Render trước, data sau → flicker | `OnInitializedAsync` |
| Sync HTTP call (`.Result`) trong WASM | Deadlock UI thread | `await` async |
| Heavy logic trong `@code` block markup | Khó test, mix UI/logic | Code-behind `.razor.cs` |
| Quên `Dispose` event subscription | Memory leak (component tree càng dùng càng leak) | `IDisposable` + unsubscribe |
| Render long list không virtualize | Slow render >500 items | `<Virtualize Items="@list">` |
| State trong static field | Shared accidentally giữa users (Server) | Scoped DI service |
| Direct DOM manipulation thay JS interop | Không sync với Blazor render | `IJSRuntime.InvokeVoidAsync` |
| Quên `StateHasChanged()` sau external event | UI không update | Gọi `StateHasChanged()` (hoặc `InvokeAsync(StateHasChanged)` từ thread khác) |

## Performance

```razor
@* Virtualize cho list lớn *@
<Virtualize Items="@products" Context="product">
    <ProductCard Product="@product" />
</Virtualize>

@* Skip render nếu props không đổi *@
@code {
    protected override bool ShouldRender() => _hasChanged;
}

@* Lazy load assembly (WASM) *@
<Router AppAssembly="@typeof(App).Assembly"
        AdditionalAssemblies="@lazyLoadedAssemblies"
        OnNavigateAsync="OnNavigateAsync">
</Router>
```

## Validate

- Render mode đúng theo nhu cầu (đừng default InteractiveServer cho mọi component)
- Component có `[Parameter]` typed, dùng `EditorRequired` cho mandatory
- `OnInitializedAsync` cho data fetch, `OnAfterRenderAsync(firstRender)` cho JS interop
- `EditForm` + validator cho mọi form (không tự handle bằng `@onsubmit`)
- Service DI scoped đúng — không static field cho user state
- `IDisposable` cho subscription, timer
- `<Virtualize>` cho list >100 items
- Cascade values chỉ cho data thực sự cross-cutting (theme, user) — không nghiệp vụ

## Hand off to

- Auth pipeline / JWT / cookie config → `aspnet-core`
- Server-side data → `entity-framework-core`
- Khi cần API JSON cho mobile/3rd-party → `web-api` / `minimal-apis`
- Real-time push beyond Blazor circuit (cross-user notification) → `signalr`
- C# language features → `modern-csharp`
