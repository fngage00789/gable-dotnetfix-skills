# Three-Pillar Output Format

The PLAN phase produces **three linked Markdown files** (the "3 pillars"). They share the same
issue IDs and cross-link. Never produce one without the others.

```
Pillar 1  All Tasks        → <Module>-All-Tasks.md          (every issue: triage + status board, merged)
Pillar 2  Frontend plan     → UAT-<FEStack>-Plan-<Module>.md (Before/After per FE issue)
Pillar 3  Backend plan      → UAT-<BEStack>-Plan-<Module>.md (Before/After per BE issue)
```

- `<Module>` = short module name from the sheet, e.g. `OPEX-Maintenance`.
- `<FEStack>` / `<BEStack>` = the **auto-detected** stack name (see below), e.g. `Angular`, `Dotnet`.
- Split by **coverage, not doc-type**: one all-tasks file + one plan per stack.
- A **fullstack** issue appears **once** in All-Tasks and in **both** plans — each plan shows only
  that layer's half (its own Before/After).

---

## Stack auto-detection (run before writing pillars 2 & 3)

Inspect the repo to name the stacks and fill each plan's "Stack:" header — **never hardcode versions**.

| Probe | Reads | Yields |
|---|---|---|
| `package.json` | `@angular/core` version, `primeng`, `tailwindcss`, `@ngrx/*` | `FEStack = Angular` + libs/version |
| `angular.json` present | — | confirms Angular workspace |
| `*.csproj` / `global.json` | `<TargetFramework>` (e.g. `net8.0`), `EntityFrameworkCore`, `MediatR` | `BEStack = Dotnet` + version |
| `*.sql` / `CREATE PROC` files | stored-proc names | enables the 🗄️ SQL owner |

- Fall back for FE: React / Vue from deps if no Angular.
- Plan **filenames** and the **"Stack:"** line both derive from what is detected
  (`UAT-Angular-Plan-OPEX-Maintenance.md`, `UAT-Dotnet-Plan-OPEX-Maintenance.md`).
- If only one layer has issues, only that plan is produced — but All-Tasks is always produced.

---

## The Before/After contract (the whole point of the plans)

Pillars 2 & 3 are worthless if the code blocks are invented. **Read the real file from the working
tree** before writing any Before block.

1. `Grep`/`Read` the target file to find the exact current code → that verbatim code is the **Before** block.
2. Write the **After** block as the minimal real edit, with a trailing `// ← FIXED` / `<!-- ← FIXED -->`
   on each changed line and `// ← BUG:` on the Before lines that are wrong.
3. Cite the file + line: `> Owner: 🅰️ Angular · \`file.ts:754\``.
4. If you genuinely cannot locate the exact line, mark the block `*(approximate)*` and say why.
   Never silently guess — an unmarked wrong Before block is the worst failure mode.
5. Keep blocks short — only the lines that change plus 1-2 for context. Use the real language fence
   (`typescript`, `html`, `scss`, `csharp`, `sql`).
6. After a fix ships, update the block with a **Fixed (date · commit `sha`)** note describing what
   actually shipped (often differs from the planned snippet — say so), tick the checkbox in the plan
   AND in All-Tasks, and update the All-Tasks status + summary counts.

---

## Pillar 1 — All Tasks (`<Module>-All-Tasks.md`)

Merged triage + status board: every tracked issue, classified, with its root cause and live status.

````markdown
# UAT Tasks — <Module>

> Source: `<excel file>` · synced **<date>** · <N> tracked issues
> **Codebases (auto-detected):**
> - Frontend — `<path>/` — <FEStack> <ver>, <libs>
> - Backend — `<path>/` — <BEStack> <ver>, <SQL if any>
> **Owner legend:** 🅰️ Angular · 🔷 .NET · 🗄️ SQL/DB · ❓ Needs clarification

## Summary

| Status | Count |   | Layer | Count |
|--------|-------|---|-------|-------|
| 🟢 Fixed | <n> |  | FRONTEND  | <n> |
| 🟡 Inprogress | <n> | | BACKEND | <n> |
| 🔴 Open | <n> |   | FULLSTACK | <n> |
| 🟠 Reopen / 🔵 Retest / 🔎 Investigate | ... | | | |

| BMS | Layer | Status | Owner | Root cause (one line) |
|-----|-------|--------|-------|------------------------|
| BMS31 | FE | Fixed | 🅰️ | <one-line root cause> |
| ...   | ... | ...   | ... | ... |

## Frontend — UI / Design / <FEStack>

### <emoji> BMS<id> | <Priority> | <Type> | Dev: <name> | <Status>
> Fix: <date> | <sync note, e.g. "Sync (3): Open → Fixed">

**Issue:**
- <original issue text, VERBATIM — keep source language, e.g. Thai>

**Remark (UAT):**
- <reviewer remark if present>

**G-ABLE Note:** <root cause + what shipped, in English; link to the relevant plan file/section>

- [ ] Fix implemented
- [ ] Unit/integration test written
- [ ] UAT retest passed

## Backend — Logic / API / SQL
## Fullstack — Both layers
<same per-issue shape; fullstack issues listed once here, with a note that the FE half lives in the
 FE plan and the BE half in the BE plan>
````

Rules:
- Every tracked issue appears **exactly once** (in Frontend, Backend, or Fullstack), plus one Summary row.
- "Root cause (one line)" in the Summary must match the G-ABLE Note's longer explanation.
- Original issue text stays **verbatim in its source language**; English analysis goes only in G-ABLE Note.
- Mark `*(repro)*` issues needing live reproduction and `*(new)*` issues new to this sync.
- Status emoji: 🟢 Fixed · 🟡 Inprogress · 🔴 Open · 🟠 Reopen · 🔵 Retest · 🔎 Investigate.

---

## Pillars 2 & 3 — By-stack Execution Plans (`UAT-<Stack>-Plan-<Module>.md`)

**One template, parameterized by the detected stack.** The FE plan uses `<FEStack>`/`ng build`;
the BE plan uses `<BEStack>`/`dotnet build` and includes SQL stored-proc changes when detected.
A fullstack issue appears in both plans, each showing only its own layer's Before/After.

````markdown
# UAT <Stack> Execution Plan — <Module>

**Module:** <Module> | **Stack:** <Stack> <ver> (<libs>) | **Status filter:** Open + Reopen | **Date:** <date>

> Source: [<Module>-All-Tasks.md](<Module>-All-Tasks.md) · `<layer path>/`.
> **Before/After blocks show real code** read from the working tree; line numbers cited per task.
> Blocks marked *(approximate)* are best-guess where the exact line wasn't found.

## Summary
| Status | Count |        | Phase | Theme | Count |
|--------|-------|        |-------|-------|-------|
| 🟢 Fixed | <n> |        | 1 | Wording & popup text | <n> |
| 🟡 Partial | <n> |      | 2 | Validation / logic guard | <n> |
| 🔴 Open | <n> |         | 3 | Layout & CSS (FE) / SQL & data (BE) | <n> |
| **Total** | **<n>** |   | 4 | State & logic (reproduce first) | <n> |

## Suggested Next Session
<short recommended order: lowest-risk first, defer items needing live reproduction>

## Phase 0 — Setup (once)
- [ ] FE: `cd <fe path>` · `npm install` · `ng build` (clean baseline) · branch `fix/...`
- [ ] BE: `cd <be path>` · `dotnet restore` · `dotnet build` (clean baseline) · branch `fix/...`

## Phase 1 — <Theme>
### <emoji> BMS<id> | <Status> — <short title>
> Owner: 🅰️ Angular | 🔷 .NET | 🗄️ SQL · `file.ext:<line>`
> <one-line root note>

**Before:**
```typescript
// file.ts — <context>
oldCode(); // ← BUG: <why wrong>
```
**After:**
```typescript
newCode(); // ← FIXED
```

- [ ] Fix implemented
- [ ] Unit/integration test written
- [ ] UAT retest passed

## Phase 5 — Final Verify
- [ ] Build (zero errors) + lint  ·  walk each item Before → After against the UAT description
- [ ] Re-confirm *(approximate)* snippets against the live file before committing
````

- Group issues into phases by **risk and shared files** — lowest-risk wording/CSS first, logic last.
- FE plan phases lean Layout/CSS/validation-styling; BE plan phases lean SQL/data/handler logic.
- Use the stack's real build command in Phase 0 / Final Verify (`ng build` vs `dotnet build`).

---

## Cross-pillar consistency (check before finishing)

- Same set of issue IDs across all files; each issue's **status is identical** everywhere.
- A fix → update the Before/After block (Fixed note) → tick the box in the plan → tick the box in
  All-Tasks → flip the All-Tasks status + Summary counts.
- Fullstack issue = once in All-Tasks (Fullstack section) + a FE half in the FE plan + a BE half in
  the BE plan. Don't duplicate the same Before/After across both plans.
- Every Before block in a plan was read from the real tree, or is explicitly `*(approximate)*`.
- Sync-discrepancy issues (sheet says Open but code is fixed) are flagged in all files.
