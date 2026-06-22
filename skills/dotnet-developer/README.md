# dotnet-developer

A **Claude Code Agent Skill**: a senior .NET developer that defaults to .NET 8, writes
compilable code, and always follows a fix with tests. It routes across C#, ASP.NET Core,
EF Core, Azure, Clean Architecture/CQRS, Blazor, DevOps, testing, security, and
observability — loading focused reference docs on demand.

> Installed the same way as other directory-listed skills such as
> [`angular/angular-developer`](https://claudemarketplaces.com/skills/angular/skills/angular-developer).

## Prerequisites

You install skills with the [`skills`](https://github.com/vercel-labs/skills) CLI, run through
`npx`. You need:

- **[Node.js](https://nodejs.org) ≥ 16.7** — provides `npx` (check with `node -v`).
- **[Claude Code](https://claude.com/claude-code)** (or another supported agent: Cursor, Codex,
  OpenCode, etc.).

No `npm install` or publish step is required — GitHub is the registry.

## Install

```bash
# install just this skill
npx -y skills add fngage00789/gable-dotnetfix-skills --skill dotnet-developer --agent claude-code
```

```bash
# or install every skill in the repo
npx -y skills add fngage00789/gable-dotnetfix-skills --agent claude-code
```

```bash
# see what's available without installing
npx -y skills add fngage00789/gable-dotnetfix-skills --list
```

This copies the skill into your agent's skills directory (`.claude/skills/dotnet-developer/`
for Claude Code). Use `-a cursor` / `-a codex` / `-a opencode` to target a different agent.

## Use it

Restart Claude Code (or run `/skills`), then invoke:

```
/dotnet-developer
```

…or just describe a .NET task ("write unit tests for this service", "design a Minimal API",
"fix this EF Core N+1") and the skill triggers automatically on its keywords.

## Manage

```bash
npx -y skills list      # show installed skills
npx -y skills update    # pull the latest version
```

## What's inside

| File | Covers |
|------|--------|
| `SKILL.md` | Router + rules + the after-every-fix test-generation workflow (.NET 8) |
| `references/csharp-patterns.md` | async/await, DI lifetimes, `Result<T>`, primary constructors, AOT JSON |
| `references/aspnetcore-patterns.md` | Minimal API, middleware order, `IExceptionHandler`, validation, versioning |
| `references/data-patterns.md` | EF Core config, N+1 fixes, migrations, transactions, bulk ops, Dapper |
| `references/azure-patterns.md` | Functions (isolated), Service Bus, Key Vault, managed identity, Blob, AKS |
| `references/architecture-patterns.md` | Clean Architecture, CQRS + MediatR, pipeline behaviors, Outbox |
| `references/blazor-patterns.md` | Hosting/render modes, EditForm, SignalR, performance |
| `references/devops-patterns.md` | Multi-stage Docker, GitHub Actions, k8s, health checks |
| `references/security-patterns.md` | JWT, RBAC policies, OWASP defenses, security headers, CORS |
| `references/observability-patterns.md` | Serilog, OpenTelemetry, .NET Aspire, caching |

## License

[MIT](../../LICENSE)
