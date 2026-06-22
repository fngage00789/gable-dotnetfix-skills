# EF Core / data patterns

## DbContext + config
```csharp
public class AppDbContext(DbContextOptions<AppDbContext> opt) : DbContext(opt)
{
    public DbSet<Budget> Budgets => Set<Budget>();
    protected override void OnModelCreating(ModelBuilder mb) =>
        mb.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
}

public class BudgetConfig : IEntityTypeConfiguration<Budget>
{
    public void Configure(EntityTypeBuilder<Budget> b)
    {
        b.HasKey(x => x.Id);
        b.Property(x => x.Name).HasMaxLength(200).IsRequired();
        b.Property(x => x.Amount).HasPrecision(18, 2);
        b.HasIndex(x => x.Year);
    }
}
```

## N+1 — the #1 perf bug
```csharp
// BAD: query per item in loop
var orders = await _ctx.Orders.ToListAsync(ct);
foreach (var o in orders) Console.WriteLine(o.Customer.Name); // lazy hit each row

// GOOD: eager load
var orders = await _ctx.Orders.Include(o => o.Customer).ToListAsync(ct);

// BEST for reads: project to DTO, no tracking, only needed cols
var dtos = await _ctx.Orders
    .AsNoTracking()
    .Select(o => new OrderDto(o.Id, o.Customer.Name, o.Total))
    .ToListAsync(ct);
```

## read vs write
- reads → `AsNoTracking()` (no change-tracker overhead).
- always `Select` to DTO; never return entities over the wire.
- `AsSplitQuery()` when multiple collection `Include`s cause cartesian explosion.

## migrations
```bash
dotnet ef migrations add AddBudgetYear -p Infrastructure -s Api
dotnet ef database update -p Infrastructure -s Api
dotnet ef migrations script --idempotent -o migrate.sql   # for prod/CI
```
- never edit applied migrations. add a new one.
- apply via script in prod, not `MigrateAsync()` on startup (race in multi-instance).

## transactions
```csharp
await using var tx = await _ctx.Database.BeginTransactionAsync(ct);
try { /* multiple SaveChanges */ await tx.CommitAsync(ct); }
catch { await tx.RollbackAsync(ct); throw; }
```
- single `SaveChangesAsync` is already atomic — no explicit tx needed.

## bulk
```csharp
// EF Core 7+ — set-based, no load into memory
await _ctx.Budgets.Where(b => b.Year < 2020)
    .ExecuteUpdateAsync(s => s.SetProperty(b => b.Archived, true), ct);
await _ctx.Budgets.Where(b => b.Archived).ExecuteDeleteAsync(ct);
```

## Dapper (hot read paths / complex SQL)
```csharp
const string sql = "SELECT id, name, amount FROM budgets WHERE year = @year";
var rows = await conn.QueryAsync<BudgetDto>(new CommandDefinition(sql, new { year }, cancellationToken: ct));
```
- parameterize always — never string-concat (SQL injection).
- EF for writes/domain, Dapper for read-heavy reporting.

## concurrency
```csharp
b.Property(x => x.RowVersion).IsRowVersion();   // optimistic; catch DbUpdateConcurrencyException
```
