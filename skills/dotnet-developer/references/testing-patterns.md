# Testing patterns

## unit (xUnit + Moq + FluentAssertions)
```csharp
public class BudgetServiceTests
{
    private readonly Mock<IBudgetRepository> _repo = new();
    private readonly BudgetService _sut;
    public BudgetServiceTests() => _sut = new(_repo.Object);

    [Fact]
    public async Task GetAsync_Exists_ReturnsDto()
    {
        _repo.Setup(r => r.FindAsync(1, default)).ReturnsAsync(new Budget { Id=1, Name="Q1" });
        var r = await _sut.GetAsync(1, default);
        r!.Name.Should().Be("Q1");
    }

    [Fact]
    public async Task GetAsync_Missing_ReturnsNull()
    {
        _repo.Setup(r => r.FindAsync(99, default)).ReturnsAsync((Budget?)null);
        (await _sut.GetAsync(99, default)).Should().BeNull();
    }

    [Fact]
    public async Task CreateAsync_BadAmount_Throws()
        => await FluentActions.Invoking(() => _sut.CreateAsync(new(-1,"x"), default))
            .Should().ThrowAsync<ValidationException>();
}
```

## playwright API (no browser)
```csharp
public class BudgetPlaywrightTests : IAsyncLifetime
{
    private IPlaywright _pw = null!;
    private IAPIRequestContext _api = null!;

    public async Task InitializeAsync()
    {
        _pw = await Playwright.CreateAsync();
        _api = await _pw.APIRequest.NewContextAsync(new() {
            BaseURL = "https://localhost:5001",
            IgnoreHTTPSErrors = true,
            ExtraHTTPHeaders = new() { ["Authorization"] = $"Bearer {await GetTokenAsync()}" }
        });
    }
    public async Task DisposeAsync() { await _api.DisposeAsync(); _pw.Dispose(); }

    [Fact] public async Task Get_200() =>
        (await _api.GetAsync("/api/budgets/1")).Status.Should().Be(200);

    [Fact] public async Task Get_404() =>
        (await _api.GetAsync("/api/budgets/99999")).Status.Should().Be(404);

    [Fact] public async Task Post_201()
    {
        var r = await _api.PostAsync("/api/budgets", new() {
            DataObject = new { name="Test", amount=1000, year=2026 }
        });
        r.Status.Should().Be(201);
        r.Headers["location"].Should().Contain("/api/budgets/");
    }

    [Fact] public async Task Post_BadAmount_400() =>
        (await _api.PostAsync("/api/budgets", new() { DataObject = new { name="x", amount=-1 } }))
            .Status.Should().Be(400);

    [Fact] public async Task Put_200() =>
        (await _api.PutAsync("/api/budgets/1", new() { DataObject = new { name="Updated", amount=2000 } }))
            .Status.Should().Be(200);

    [Fact] public async Task Delete_204() =>
        (await _api.DeleteAsync("/api/budgets/1")).Status.Should().Be(204);

    static Task<string> GetTokenAsync() => Task.FromResult("test-jwt-token");
}

// reusable factory
public static class ApiFactory
{
    public static async Task<IAPIRequestContext> CreateAsync(string baseUrl, string? token = null)
    {
        var pw = await Playwright.CreateAsync();
        return await pw.APIRequest.NewContextAsync(new() {
            BaseURL = baseUrl, IgnoreHTTPSErrors = true,
            ExtraHTTPHeaders = token is null ? new() : new() { ["Authorization"] = $"Bearer {token}" }
        });
    }
}
```

## integration (WebApplicationFactory + Testcontainers)
```csharp
public class BudgetIntegrationTests : IAsyncLifetime
{
    private readonly PostgreSqlContainer _db = new PostgreSqlBuilder().Build();
    private WebApplicationFactory<Program> _factory = null!;

    public async Task InitializeAsync()
    {
        await _db.StartAsync();
        _factory = new WebApplicationFactory<Program>().WithWebHostBuilder(b =>
            b.ConfigureServices(s => {
                s.RemoveAll<DbContextOptions<AppDbContext>>();
                s.AddDbContext<AppDbContext>(o => o.UseNpgsql(_db.GetConnectionString()));
            }));
        using var scope = _factory.Services.CreateScope();
        await scope.ServiceProvider.GetRequiredService<AppDbContext>().Database.MigrateAsync();
    }
    public async Task DisposeAsync() { await _factory.DisposeAsync(); await _db.DisposeAsync(); }

    [Fact] public async Task Post_PersistsToDb()
    {
        var r = await _factory.CreateClient().PostAsJsonAsync("/api/budgets",
            new { name="Test", amount=500, year=2026 });
        r.StatusCode.Should().Be(HttpStatusCode.Created);
        using var s = _factory.Services.CreateScope();
        s.ServiceProvider.GetRequiredService<AppDbContext>().Budgets
            .Should().Contain(b => b.Name == "Test");
    }
}
```

## arch fitness (NetArchTest)
```csharp
[Fact] public void Domain_NoDep_Infrastructure() =>
    Types.InAssembly(typeof(Budget).Assembly)
        .Should().NotHaveDependencyOn("BMS.Infrastructure")
        .GetResult().IsSuccessful.Should().BeTrue();
```

## load (k6)
```js
import http from 'k6/http'; import { check } from 'k6';
export const options = { stages:[{duration:'30s',target:50}],
    thresholds:{http_req_duration:['p(95)<500'],http_req_failed:['rate<0.01']} };
export default () => check(http.get('http://localhost:5001/api/budgets'), {'ok':r=>r.status===200});
```
