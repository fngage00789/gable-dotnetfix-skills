---
name: excel-issue-fixer
description: >
  Read Excel issue list, classify backend (.NET/ASP.NET Core) vs frontend (Angular),
  write implementation plan, then implement. Combines dotnet-developer + angular-developer.
  Trigger: "read excel", "fix issues from excel", "excel bug list", "issue sheet", ".xlsx", "import issues".
---

# Excel Issue Fixer

Combines: `dotnet-developer` (backend) + `angular-developer` (frontend impl).

## invocation

```
/excel-issue-fixer @<path-to-file.xlsx>
```

The `@<file>` is the Excel issue sheet. If no path is given, ask for one before doing anything else.

## flow (strict order — never skip)

```
1. PARSE    → read the @<file> Excel, extract all rows
2. SELECT   → ASK the user to pick the frontend + backend projects (see ## select codebases)
3. CLASSIFY → each issue: backend | frontend | fullstack
4. DETECT   → auto-detect FE + BE stack inside the selected paths (see ## stack auto-detect)
5. PLAN     → write the 3 pillar files (All-Tasks → FE plan → BE plan), then TodoWrite
6. SUGGEST  → show Before/After per issue, then POPUP-ASK the user to choose: Review/Approve or
              Auto-fix all — never edit a file before this popup returns (see ## suggest before/after)
7. IMPLEMENT→ execute approved plan, backend first → frontend → tests
8. VERIFY   → run tests, confirm no regression; sync status across all 3 pillars
```

The PLAN output is the **3-pillar set** — see `references/three-pillar-output.md` for the full
templates and the Before/After contract.

## select codebases (after parse — mandatory)

Before classifying, **prompt the user to choose the two project roots** — never assume the cwd.
Use the `AskUserQuestion` tool (it renders as a selectable popup); one question per role:

- **Frontend project** — the Angular (or React/Vue) repo/folder. Offer detected candidates as
  options (dirs containing `package.json` + `angular.json`), plus "None / backend-only".
- **Backend project** — the .NET repo/folder. Offer detected candidates (dirs containing
  `*.csproj` / `*.sln` / `global.json`), plus "None / frontend-only".

Discover candidates first so the popup has real choices:

```bash
# frontend candidates
find . -maxdepth 4 -name angular.json -not -path '*/node_modules/*' -printf '%h\n' 2>/dev/null
# backend candidates
find . -maxdepth 4 \( -name '*.sln' -o -name '*.csproj' \) -not -path '*/bin/*' -printf '%h\n' 2>/dev/null | sort -u
```

Record the picks as `FERoot` and `BERoot`. All later steps (DETECT, plan filenames, Before/After
file reads, IMPLEMENT) operate **inside those roots**. If the user picks "None" for a role, skip
that role's plan pillar entirely.

## suggest before/after (HARD gate — popup before any code)

After PLAN, **present the Before/After suggestion for each issue**, then **STOP and ask via a popup**.
This is a hard gate — exactly like SELECT, it must use the `AskUserQuestion` tool. **No file may be
edited until that popup returns.** Do not infer approval from the original prompt; the only way past
this gate is the user's answer in *this* popup.

1. Show, per issue: `id · title`, the **Before** block (real code read from `FERoot`/`BERoot`,
   verbatim with `file:line`) and the proposed **After** block (`// ← FIXED`).
   Group by codebase (Backend first, then Frontend). Keep it skimmable — diff-style.

2. Then call `AskUserQuestion` with these options:
   - **Auto-fix all** — implement every suggested fix now, no further prompts.
   - **Review / approve subset** — user names which issue ids to apply; only those get implemented.
   - **Just write the plan** — stop here, edit nothing; the 3-pillar plan files are the deliverable.
   - **Request changes** — user gives feedback; revise the Before/After and re-show, then re-ask.

3. Only after the popup returns do you proceed to IMPLEMENT, and only for the approved scope.
   Re-show / re-ask after edits if the user picked "Review / approve subset".

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

Inspect the selected `FERoot` / `BERoot` — never hardcode versions. Fill the "Codebases"/"Stack:"
headers from what you find.

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
- **Pillar 1 is the sheet-Status board — not a free-form triage.** Drive every status from the Excel
  **Status column (col H)**; apply the done-bucket rule (`Retest UAT` ≡ `Fixed` ≡ `Close` = done);
  group per-issue sections by **layer** (Frontend / Backend / Fullstack). Pillar 1 MUST contain, in
  order: header line with the Status-column breakdown + which export `(NN)`, a **Sync log**, a
  **Summary** (status table + layer table), a **Tasks Left** section grouped by owner, a **Triage**
  table, then the per-layer per-issue sections. **Never** restructure Pillar 1 around an invented
  P1/P2/REPRO/BIZ priority — risk/priority phasing belongs only in the plans (Pillars 2 & 3).
- **Before/After contract:** the Before block is the **real code read from `FERoot`/`BERoot`** (verbatim,
  `Grep`/`Read` it first); After is the minimal edit with `// ← FIXED` / `// ← BUG:` markers; cite `file:line`
  as a clickable link; mark `*(approximate)*` only when the exact line can't be found — never silently guess.
- Each plan issue carries 3 checkboxes: Fix implemented · Unit/integration test written · UAT retest passed.
- **Re-runs accumulate, never reset:** if a `<Module>-All-Tasks.md` already exists, update it in place —
  append a new Sync-log delta and flip changed statuses/counts. Do not regenerate it from scratch.

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
- After impl → Angular unit/spec tests via `ng test`

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
frontend → src/app/**/{feature}.spec.ts   # Angular ng test (Karma/Jasmine or Jest)
```

## scaffold check (run once if missing)

```bash
# backend test deps
dotnet add package xunit Moq FluentAssertions
dotnet add package Microsoft.AspNetCore.Mvc.Testing Testcontainers.PostgreSql

# frontend tests — Angular CLI ships its own test runner
ng test --watch=false

# excel parsing
pip install pandas openpyxl
```
