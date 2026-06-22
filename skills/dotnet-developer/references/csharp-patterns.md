# C# patterns

## async — rules
- `async Task` not `async void`.
- never `.Result` / `.Wait()` → deadlock.
- `CancellationToken` on all I/O.
- DI lifetimes: Transient=new each time, Scoped=per request, Singleton=app-life (thread-safe!).

## Result<T>
```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public string? Error { get; }
    private Result(T v) { IsSuccess = true; Value = v; }
    private Result(string e) { Error = e; }
    public static Result<T> Ok(T v) => new(v);
    public static Result<T> Fail(string e) => new(e);
}
```

## primary constructors (.NET 8+)
```csharp
public class UserService(IUserRepository repo, ILogger<UserService> logger)
{
    public async Task<UserDto?> GetAsync(int id, CancellationToken ct)
    {
        logger.LogInformation("Getting {Id}", id);
        return await repo.FindAsync(id, ct) is { } u ? u.ToDto() : null;
    }
}
```

## channels (producer/consumer)
```csharp
var ch = Channel.CreateBounded<WorkItem>(100);
await ch.Writer.WriteAsync(item, ct);
await foreach (var item in ch.Reader.ReadAllAsync(ct))
    await ProcessAsync(item, ct);
```

## AOT-safe JSON
```csharp
[JsonSerializable(typeof(UserDto))]
internal partial class AppJsonContext : JsonSerializerContext { }
builder.Services.ConfigureHttpJsonOptions(o =>
    o.SerializerOptions.TypeInfoResolverChain.Insert(0, AppJsonContext.Default));
```
