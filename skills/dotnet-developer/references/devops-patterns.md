# DevOps patterns

## Dockerfile (multi-stage, non-root, .NET 8)
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.sln ./
COPY src/Api/*.csproj src/Api/
RUN dotnet restore src/Api/Api.csproj
COPY . .
RUN dotnet publish src/Api/Api.csproj -c Release -o /app --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=build /app .
USER $APP_UID                          # built-in non-root user (.NET 8)
ENTRYPOINT ["dotnet", "Api.dll"]
```
- restore csproj before copying all → layer cache on dependency changes only.
- chiseled/distroless base for smaller attack surface: `aspnet:8.0-jammy-chiseled`.

## GitHub Actions CI
```yaml
name: ci
on: { push: { branches: [main] }, pull_request: {} }
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with: { dotnet-version: '8.0.x' }
      - run: dotnet restore
      - run: dotnet build --no-restore -c Release
      - run: dotnet test --no-build -c Release --collect:"XPlat Code Coverage"
      - uses: actions/upload-artifact@v4
        with: { name: coverage, path: '**/coverage.cobertura.xml' }
```

## health checks
```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>("db")
    .AddNpgSql(conn, name: "postgres");
app.MapHealthChecks("/health/live", new() { Predicate = _ => false });   // liveness: process up
app.MapHealthChecks("/health/ready");                                     // readiness: deps ok
```

## k8s deployment (essentials)
```yaml
spec:
  containers:
    - name: api
      image: registry/api:${TAG}
      resources:
        requests: { cpu: 100m, memory: 128Mi }
        limits:   { cpu: 500m, memory: 256Mi }
      livenessProbe:  { httpGet: { path: /health/live,  port: 8080 } }
      readinessProbe: { httpGet: { path: /health/ready, port: 8080 } }
```
- pin image by digest/tag, never `:latest` in prod.
- HPA on CPU/memory or KEDA on queue length.

## release hygiene
- semantic version + tag. immutable artifacts.
- migrations as idempotent script step before deploy, gated.
- rollback = redeploy previous image; keep DB changes backward-compatible.
