# Blazor patterns

## hosting model — pick
| model | latency | offline | SEO | use when |
|-------|---------|---------|-----|----------|
| Server | low (SignalR rtt) | no | yes | internal apps, fast first load |
| WASM | zero after load | yes | needs prerender | public, offline, scale out |
| Auto (.NET 8) | best of both | partial | yes | default for new apps |

## component basics
```razor
@inject IBudgetService Svc
@implements IDisposable

<h3>@Title</h3>
@if (_loading) { <p>loading…</p> }
else { @foreach (var b in _items) { <div>@b.Name — @b.Amount</div> } }

@code {
    [Parameter] public string Title { get; set; } = "Budgets";
    private List<BudgetDto> _items = [];
    private bool _loading = true;
    private CancellationTokenSource _cts = new();

    protected override async Task OnInitializedAsync()
    {
        _items = await Svc.GetAllAsync(_cts.Token);
        _loading = false;
    }
    public void Dispose() { _cts.Cancel(); _cts.Dispose(); }
}
```

## render modes (.NET 8)
```razor
@rendermode InteractiveServer      // per-component
@rendermode InteractiveWebAssembly
@rendermode InteractiveAuto
```
- static SSR by default — add render mode only where interactivity needed.

## EditForm + validation
```razor
<EditForm Model="_model" OnValidSubmit="Save">
    <DataAnnotationsValidator />
    <InputText @bind-Value="_model.Name" />
    <ValidationMessage For="() => _model.Name" />
    <button type="submit">Save</button>
</EditForm>
```

## state + re-render
- `StateHasChanged()` only when mutating outside an event (timer, SignalR push).
- `@key` on list items to keep identity across reorders.
- avoid heavy work in `OnParametersSet` (runs every render).

## SignalR hub
```csharp
public class BudgetHub : Hub
{
    public async Task Subscribe(string year) =>
        await Groups.AddToGroupAsync(Context.ConnectionId, year);
}
// server push: await _hub.Clients.Group(year).SendAsync("Updated", dto, ct);
```

## perf
- `@rendermode InteractiveServer` → circuit per user; watch memory at scale.
- WASM → trim + AOT for size/speed; lazy-load assemblies per route.
- virtualize big lists: `<Virtualize Items="_items" Context="b">`.
