# Azure patterns

## Functions (isolated worker, .NET 8)
```csharp
public class BudgetFn(ILogger<BudgetFn> log)
{
    [Function("ProcessBudget")]
    public async Task Run([ServiceBusTrigger("budgets", Connection = "SbConn")] string msg,
        CancellationToken ct)
    {
        log.LogInformation("got {Msg}", msg);
        // ...
    }

    [Function("GetBudget")]
    public async Task<HttpResponseData> Http(
        [HttpTrigger(AuthorizationLevel.Function, "get", Route = "budgets/{id}")] HttpRequestData req,
        int id, CancellationToken ct)
    {
        var res = req.CreateResponse(HttpStatusCode.OK);
        await res.WriteAsJsonAsync(new { id }, ct);
        return res;
    }
}
```
- isolated worker > in-process (decoupled runtime, .NET 8+ requires it).

## Service Bus (sender)
```csharp
await using var client = new ServiceBusClient(conn, new DefaultAzureCredential());
var sender = client.CreateSender("budgets");
await sender.SendMessageAsync(new ServiceBusMessage(BinaryData.FromObjectAsJson(evt)) {
    MessageId = id.ToString(),          // dedup
    SessionId = tenantId                 // FIFO per session
}, ct);
```
- queue = point-to-point. topic+subscription = pub/sub.
- handle poison: max delivery count → dead-letter queue.

## Key Vault — secrets, never appsettings
```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri($"https://{vaultName}.vault.azure.net/"),
    new DefaultAzureCredential());
// then read like normal config: builder.Configuration["DbPassword"]
```

## managed identity (no connection strings/keys)
```csharp
var cred = new DefaultAzureCredential();   // works local (az login) + cloud (MI)
var blob = new BlobServiceClient(new Uri($"https://{acct}.blob.core.windows.net"), cred);
```
- DefaultAzureCredential chain: env → managed identity → az cli. same code everywhere.

## Blob storage
```csharp
var container = blob.GetBlobContainerClient("uploads");
await container.CreateIfNotExistsAsync(cancellationToken: ct);
await container.GetBlobClient(name).UploadAsync(stream, overwrite: true, ct);
```

## AKS deploy notes
- liveness/readiness probes → `/health/live`, `/health/ready`.
- requests/limits set; HPA on CPU or custom metric.
- secrets via Key Vault CSI driver or Workload Identity, not k8s Secrets in git.

## config precedence (later wins)
appsettings.json → appsettings.{Env}.json → user-secrets (dev) → env vars → Key Vault.
