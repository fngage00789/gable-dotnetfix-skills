---
name: excel-issue-fixer
description: >
  Read Excel issue list, classify backend (.NET/ASP.NET Core) vs frontend (Angular),
  write implementation plan, then implement. Combines dotnet-developer + angular-developer + playwright-angular-qa.
  Trigger: "read excel", "fix issues from excel", "excel bug list", "issue sheet", ".xlsx", "import issues".
---

# Excel Issue Fixer

Combines: `dotnet-developer` (backend) + `angular-developer` (frontend impl) + `playwright-angular-qa` (e2e tests).

## flow (strict order — never skip)

```
1. PARSE    → read Excel, extract all rows
2. CLASSIFY → each issue: backend | frontend | fullstack
3. DETECT   → auto-detect FE + BE stack from the repo (see ## stack auto-detect)
4. PLAN     → write the 3 pillar files (All-Tasks → FE plan → BE plan), then TodoWrite
5. CONFIRM  → show plan, wait for approval (or auto if told)
6. IMPLEMENT→ execute plan, backend first → frontend → tests
7. VERIFY   → run tests, confirm no regression; sync status across all 3 pillars
```

The PLAN output is the **3-pillar set** — see `references/three-pillar-output.md` for the full
templates and the Before/After contract.

## parse rules

```python
import pandas as pd
df = pd.read_excel("<file>")
# expected columns (flexible, match by keyword):
# ID / No / # | Title / Issue / Description | Type / Category | Priority / Severity
# if columns differ → infer from content
```

Output per row:
```
{ id, title, description, priority, raw_type }
```

## classify rules

| Signal words | Classify |
|---|---|
| API, endpoint, controller, service, repository, DbContext, migration, EF, SQL, auth, JWT, middleware, CQRS, MediatR, command, query, handler | **backend** |
| component, template, HTML, CSS, SCSS, Angular, routing, form, reactive form, ngModel, HTTP client, interceptor, pipe, directive, UI, UX, style | **frontend** |
| both signals present, or "full stack", or unclear | **fullstack** |

## stack auto-detect (run before writing the plans)

Inspect the repo — never hardcode versions. Fill the "Codebases"/"Stack:" headers from what you find.

| Probe | Reads | Yields |
|---|---|---|
| `package.json` | `@angular/core` ver, `primeng`, `tailwindcss`, `@ngrx/*` | `FEStack = Angular` + libs/ver |
| `angular.json` present | — | confirms Angular workspace |
| `*.csproj` / `global.json` | `<TargetFramework>`, `EntityFrameworkCore`, `MediatR` | `BEStack = Dotnet` + ver |
| `*.sql` / `CREATE PROC` | stored-proc names | enables 🗄️ SQL owner |

FE fall back: React / Vue from deps. Plan **filenames** + the **"Stack:"** line derive from what's
detected (`UAT-Angular-Plan-<Module>.md`, `UAT-Dotnet-Plan-<Module>.md`).

## plan = 3 pillars (write before any code)

Produce three linked files (full templates in `references/three-pillar-output.md`):

```
Pillar 1  All Tasks    → <Module>-All-Tasks.md          every issue (triage + status board, merged),
                                                          grouped by layer, verbatim issue text + root cause
Pillar 2  Frontend plan → UAT-<FEStack>-Plan-<Module>.md  Before/After per FE issue, phased by risk
Pillar 3  Backend plan  → UAT-<BEStack>-Plan-<Module>.md  Before/After per BE issue, phased
```

- Split by **coverage, not doc-type**: one all-tasks file + one plan per stack.
- A **fullstack** issue → once in All-Tasks, and in **both** plans (each shows only its layer's half).
- **Before/After contract:** the Before block is the **real code read from the working tree** (verbatim,
  `Grep`/`Read` it first); After is the minimal edit with `// ← FIXED` / `// ← BUG:` markers; cite `file:line`;
  mark `*(approximate)*` only when the exact line can't be found — never silently guess.
- Each plan issue carries 3 checkboxes: Fix implemented · Unit/integration test written · UAT retest passed.

Then use TodoWrite. One todo per issue.

## implement rules

### backend issues → follow dotnet-developer
- .NET 9, primary constructors, async/await, CancellationToken
- Clean Arch: Domain → Application → Infrastructure → Api
- CQRS + MediatR for commands/queries
- xUnit + Moq tests after every fix
- never .Result / .Wait() / async void / secrets in appsettings.json

### frontend issues → follow angular-developer
- Check Angular version first (`ng version`) before generating any code
- Use latest Angular signals (`signal`, `computed`, `linkedSignal`, `resource`) for state
- Signal Forms for new forms (Angular v21+), Reactive Forms for existing
- Standalone components default — no NgModule unless required
- Use Angular CLI generators: `ng generate component|service|pipe|guard|route`
- Run `ng build` after every change — fix all errors before proceeding
- HTTPClient + interceptors for API calls
- Angular Aria patterns for accessible components (Accordion, Listbox, Menu, etc.)
- Tailwind CSS for styling if project uses it
- After impl → Playwright e2e tests via `playwright-angular-qa`

### fullstack issues
- backend first → frontend second → integration tests last
- the issue lives in BOTH plans — update the BE half in the BE plan and the FE half in the FE plan

### after every fix → sync the 3 pillars (mandatory)
- update that issue's Before/After block with a **Fixed (date · commit `sha`)** note (say what actually
  shipped if it differs from the planned snippet)
- tick `[x] Fix implemented` in the plan AND in `<Module>-All-Tasks.md`
- flip the All-Tasks status emoji + Summary counts; flag any sheet-vs-code discrepancy in all files

## test-gen (mandatory after every fix)

```
backend  → tests/Unit/{Feature}Tests.cs
           tests/Integration/{Feature}ApiTests.cs
frontend → tests/Playwright/{Feature}PlaywrightTests.cs
```

## scaffold check (run once if missing)

```bash
# backend test deps
dotnet add package xunit Moq FluentAssertions
dotnet add package Microsoft.AspNetCore.Mvc.Testing Testcontainers.PostgreSql

# frontend / e2e
npm install -D @playwright/test
npx playwright install

# excel parsing
pip install pandas openpyxl
```
