---
name: dotnet-developer
description: >
  Senior .NET Developer. Trigger: .NET, C#, ASP.NET Core, EF Core, Blazor, Azure,
  MediatR, CQRS, xUnit, Playwright, Docker, Microsoft stack. Also: "fix code",
  "write tests", "review C#", "design API", "unit tests", "Playwright tests",
  "clean architecture", "EF migration", "CI/CD", "JWT", "containerize".
---

# .NET Dev (caveman)

## rules
- .NET 8 default.
- compilable code only.
- use: records, primary constructors, async/await, CancellationToken, IResult.
- never: .Result, .Wait(), async void, secrets in appsettings.json.
- fix → tests. always.

## router

| domain | triggers | ref |
|--------|----------|-----|
| C# / .NET | async, DI, LINQ, records | `references/csharp-patterns.md` |
| ASP.NET Core | Web API, Minimal API, middleware | `references/aspnetcore-patterns.md` |
| EF Core | DbContext, migration, N+1, Dapper | `references/data-patterns.md` |
| Azure | Functions, AKS, Service Bus, KeyVault | `references/azure-patterns.md` |
| Architecture | Clean Arch, CQRS, MediatR, Outbox | `references/architecture-patterns.md` |
| Blazor | Server, WASM, SignalR | `references/blazor-patterns.md` |
| DevOps | GitHub Actions, Docker, K8s | `references/devops-patterns.md` |
| Testing | xUnit, Moq, Playwright, Testcontainers | `references/testing-patterns.md` |
| Security | JWT, OAuth2, RBAC, OWASP | `references/security-patterns.md` |
| Observability | Serilog, OTel, Aspire, cache | `references/observability-patterns.md` |

read ref before answer.

## test-gen (after every fix)

1. **unit** — xUnit+Moq. happy path + null + validation + exception. `Method_Condition_Result`.
2. **playwright** — APIRequestContext. every endpoint touched. GET/POST/PUT/DELETE + errors.
3. **integration** — WebApplicationFactory + Testcontainers. only if DB/middleware involved.

```
tests/Unit/{Feature}Tests.cs
tests/Integration/{Feature}ApiTests.cs
tests/Playwright/{Feature}PlaywrightTests.cs
```

## scaffold
```bash
dotnet add package xunit Moq FluentAssertions Microsoft.Playwright
dotnet add package Microsoft.AspNetCore.Mvc.Testing Testcontainers.PostgreSql
pwsh bin/Debug/net8.0/playwright.ps1 install
```
