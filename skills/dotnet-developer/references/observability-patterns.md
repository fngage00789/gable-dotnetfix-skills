# Observability patterns

## structured logging (Serilog)
```csharp
builder.Host.UseSerilog((ctx, cfg) => cfg
    .ReadFrom.Configuration(ctx.Configuration)
    .Enrich.FromLogContext()
    .Enrich.WithProperty("service", "budget-api")
    .WriteTo.Console(new RenderedCompactJsonFormatter()));   // JSON for log aggregators
```
```csharp
// structured, not interpolated — keeps queryable fields
log.LogInformation("Created budget {BudgetId} for {Year}", id, year);   // GOOD
log.LogInformation($"Created budget {id}");                              // BAD (flat string)
```
- log levels: Trace/Debug dev only; Info = business events; Warning = recoverable; Error = failed op; Critical = down.
- never log secrets, tokens, PII.

## correlation
```csharp
app.Use(async (ctx, next) => {
    var id = ctx.Request.Headers["X-Correlation-Id"].FirstOrDefault() ?? ctx.TraceIdentifier;
    using (LogContext.PushProperty("CorrelationId", id)) { await next(); }
});
```

## OpenTelemetry (traces + metrics)
```csharp
builder.Services.AddOpenTelemetry()
    .ConfigureResource(r => r.AddService("budget-api"))
    .WithTracing(t => t.AddAspNetCoreInstrumentation()
                       .AddHttpClientInstrumentation()
                       .AddEntityFrameworkCoreInstrumentation()
                       .AddOtlpExporter())
    .WithMetrics(m => m.AddAspNetCoreInstrumentation()
                       .AddRuntimeInstrumentation()
                       .AddOtlpExporter());
```

## custom metrics + activity
```csharp
static readonly Meter Meter = new("Budget.Api");
static readonly Counter<long> Created = Meter.CreateCounter<long>("budgets.created");
Created.Add(1, new KeyValuePair<string, object?>("year", year));

static readonly ActivitySource Source = new("Budget.Api");
using var act = Source.StartActivity("ApproveBudget");
act?.SetTag("budget.id", id);
```

## .NET Aspire (dev-time orchestration + dashboard)
```csharp
// AppHost
var db = builder.AddPostgres("pg").AddDatabase("budgets");
builder.AddProject<Projects.Api>("api").WithReference(db);
// ServiceDefaults wires OTel + health checks + service discovery automatically
builder.AddServiceDefaults();
```

## caching
```csharp
// in-memory
builder.Services.AddMemoryCache();
var dto = await _cache.GetOrCreateAsync(key, e => {
    e.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
    return svc.GetAsync(id, ct);
});

// distributed (Redis) — multi-instance
builder.Services.AddStackExchangeRedisCache(o => o.Configuration = cfg["Redis"]);

// HybridCache (.NET 9) — L1+L2, stampede protection
var dto = await _hybrid.GetOrCreateAsync(key, async ct => await svc.GetAsync(id, ct), token: ct);
```
- cache reads, not writes. invalidate on mutation. set TTL always.
- output caching for whole endpoints: `.CacheOutput()`.
```
