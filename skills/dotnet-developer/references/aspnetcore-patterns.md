# ASP.NET Core patterns

## minimal API + endpoint groups
```csharp
var budgets = app.MapGroup("/api/budgets").WithTags("Budgets").RequireAuthorization();

budgets.MapGet("/{id:int}", async (int id, IBudgetService svc, CancellationToken ct) =>
    await svc.GetAsync(id, ct) is { } dto ? Results.Ok(dto) : Results.NotFound());

budgets.MapPost("/", async (CreateBudgetDto dto, IBudgetService svc, CancellationToken ct) =>
{
    var id = await svc.CreateAsync(dto, ct);
    return Results.Created($"/api/budgets/{id}", new { id });
});
```
- return `IResult` (`Results.Ok`/`NotFound`/`Created`/`ValidationProblem`). never raw object.
- `int:`, `guid:` route constraints. bind `[AsParameters]` for big query sets.

## DI registration
```csharp
builder.Services.AddScoped<IBudgetService, BudgetService>();
builder.Services.AddProblemDetails();          // RFC 7807 errors
builder.Services.AddOutputCache();
builder.Services.AddRateLimiter(o => o.AddFixedWindowLimiter("api", w =>
    { w.PermitLimit = 100; w.Window = TimeSpan.FromMinutes(1); }));
```

## middleware order (matters!)
```csharp
app.UseExceptionHandler();      // 1. catch-all
app.UseHttpsRedirection();
app.UseRateLimiter();
app.UseAuthentication();        // who
app.UseAuthorization();         // can they
app.MapControllers();           // or MapGroup endpoints
```

## global exception → ProblemDetails (.NET 8 IExceptionHandler)
```csharp
public class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> log) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext ctx, Exception ex, CancellationToken ct)
    {
        log.LogError(ex, "unhandled");
        var (code, title) = ex switch {
            ValidationException => (400, "Validation failed"),
            KeyNotFoundException => (404, "Not found"),
            _ => (500, "Server error")
        };
        ctx.Response.StatusCode = code;
        await ctx.Response.WriteAsJsonAsync(new ProblemDetails { Status = code, Title = title }, ct);
        return true;
    }
}
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
```

## validation (FluentValidation, filter)
```csharp
budgets.MapPost("/", Handler).AddEndpointFilter<ValidationFilter<CreateBudgetDto>>();
```

## model binding gotchas
- record DTOs bind from JSON body automatically. one body param per endpoint.
- `[FromQuery]`, `[FromRoute]`, `[FromHeader]` only when ambiguous.
- enable nullable; non-null reference params are required by default.

## versioning
```csharp
builder.Services.AddApiVersioning(o => { o.DefaultApiVersion = new(1,0); o.ReportApiVersions = true; });
// /api/v1/budgets
```
