# Architecture patterns

## clean arch layers
```
Domain        → no deps. entities, value objects, events, interfaces.
Application   → depends Domain. commands/queries, behaviors, DTOs.
Infrastructure→ implements interfaces. EF, messaging, HTTP clients.
Api/Web       → entry point. endpoints, middleware, Program.cs.
```
dep rule: outer → inner only. never reverse.

## folder
```
src/
├── Domain/Entities/ ValueObjects/ Events/ Interfaces/
├── Application/Features/{Feature}/Commands/ Queries/
│              Behaviors/ DTOs/ Mappings/
├── Infrastructure/Persistence/ Messaging/ ExternalServices/
│                 DependencyInjection.cs
└── Api/Endpoints/ Middleware/ Program.cs
```

## CQRS + MediatR
```csharp
public record CreateOrderCommand(Guid CustomerId, List<OrderItem> Items)
    : IRequest<Result<Guid>>;

public class CreateOrderHandler(IOrderRepository repo, IPublisher pub)
    : IRequestHandler<CreateOrderCommand, Result<Guid>>
{
    public async Task<Result<Guid>> Handle(CreateOrderCommand cmd, CancellationToken ct)
    {
        var order = Order.Create(cmd.CustomerId, cmd.Items);
        await repo.AddAsync(order, ct);
        await pub.Publish(new OrderCreatedEvent(order.Id), ct);
        return Result.Ok(order.Id);
    }
}
```

## pipeline behavior (validation)
```csharp
public class ValidationBehavior<TReq, TRes>(IEnumerable<IValidator<TReq>> validators)
    : IPipelineBehavior<TReq, TRes>
{
    public async Task<TRes> Handle(TReq req, RequestHandlerDelegate<TRes> next, CancellationToken ct)
    {
        var errors = validators.SelectMany(v => v.Validate(req).Errors).Where(f => f != null).ToList();
        if (errors.Count != 0) throw new ValidationException(errors);
        return await next();
    }
}
// register:
cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
```

## outbox pattern
```csharp
// same transaction: save entity + outbox msg
_ctx.Orders.Add(order);
_ctx.OutboxMessages.Add(new OutboxMessage {
    Type = nameof(OrderCreatedEvent),
    Payload = JsonSerializer.Serialize(new OrderCreatedEvent(order.Id)),
    OccurredAt = DateTime.UtcNow
});
await _ctx.SaveChangesAsync(ct);

// background: poll + publish + mark done
var msgs = await _ctx.OutboxMessages.Where(m => m.ProcessedAt == null).Take(20).ToListAsync(ct);
foreach (var msg in msgs) {
    await sender.SendMessageAsync(new ServiceBusMessage(msg.Payload), ct);
    msg.ProcessedAt = DateTime.UtcNow;
}
await _ctx.SaveChangesAsync(ct);
```

## monolith vs microservices
- start monolith. extract only when: team>10 + clear bounded context + independent scaling needed.
