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

> **This format is prescriptive, not a suggestion.** Pillar 1 MUST be the *sheet-Status-driven board*
> described below — per-issue sections grouped by **layer** (Frontend / Backend / Fullstack), with
> status driven by the Excel **Status column**. Do **NOT** invent an alternative top-level scheme
> (e.g. grouping by an ad-hoc P1/P2/REPRO/BIZ priority). Priority/risk is a *plan* concern — it lives
> in Pillars 2 & 3 as **phases**, never as the structure of Pillar 1.

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

## Status model (the spine of Pillar 1 — read before writing anything)

Status is driven by the Excel sheet, **not** invented. Two columns matter:

- **Status column (col H)** — the **single source of truth** for each issue's status. Use its exact
  values verbatim: `Close`, `Retest UAT`, `Fixed`, `Open`, `Investigate`, `Inprogress Fix`, `Retest`.
- **"Status for G-ABLE" column (col R)** — used **only** to flag a UAT **Reopen** (an item dev-marked
  done that the reviewer re-opened). Otherwise ignored.

**Done-bucket rule (mandatory):** `Retest UAT` ≡ `Fixed` ≡ `Close` ≡ **done (dev-done)**. Only
`Open` / `Investigate` / `Inprogress Fix` / `Retest` are genuinely **remaining**. Every count,
summary, and "Tasks Left" list buckets on this rule.

**Status emoji map:** 🟢 Close / Fixed · 🔵 Retest UAT / Retest · 🔴 Open · 🔎 Investigate ·
🟡 Inprogress Fix · 🟠 Reopen (col R). Owner emoji: 🅰️ Angular · 🔷 .NET · 🗄️ SQL/DB · ❓ unclear.

**Sync deltas:** when re-running against a newer Excel export (e.g. `(9)` → `(11)`), do **not** rewrite
history. Append **one** blockquote to the **Sync log** recording only what changed (status flips, new
issues, counts), dated, naming which export is now source of truth. The board accumulates across runs —
that accumulation *is* the value; a fresh single-run board with no Sync log is the thin/wrong output.

---

## The Before/After contract (the whole point of the plans)

Pillars 2 & 3 are worthless if the code blocks are invented. **Read the real file from the working
tree** before writing any Before block.

1. `Grep`/`Read` the target file to find the exact current code → that verbatim code is the **Before** block.
2. Write the **After** block as the minimal real edit, with a trailing `// ← FIXED <id>` /
   `<!-- ← FIXED <id> -->` on each changed line and `// ← BUG:` on the Before lines that are wrong.
3. Cite the file + line as a **clickable link**:
   `[WorkflowServiceV2.cs:3318](budgetapplication-backend/src/Application/Services/WorkflowServiceV2.cs#L3318)`.
4. If you genuinely cannot locate the exact line, mark the block `*(approximate)*` and say why.
   Never silently guess — an unmarked wrong Before block is the worst failure mode.
5. Keep blocks short — only the lines that change plus 1-2 for context. Use the real language fence
   (`typescript`, `html`, `scss`, `csharp`, `sql`).
6. After a fix ships, update the block with a **Fixed (date · commit `sha`)** note describing what
   actually shipped (often differs from the planned snippet — say so), tick the checkbox in the plan
   AND in All-Tasks, and update the All-Tasks status + summary counts + Sync log.

---

## Pillar 1 — All Tasks (`<Module>-All-Tasks.md`)

The single source of truth for status. Merged triage + status board: every tracked issue, classified
by layer, with its verbatim text, root cause, and live status. **Produce every section below, in this
order.**

````markdown
# <Module> — All Tasks (Pillar 1: Triage + Status Board)

**Module:** <module name> | **Total issues:** <N> — by **Status** column (<Status1> <n> · <Status2> <n> · …) | **Date:** <YYYY-MM-DD> | **Sync:** Excel `(<NN>)`

> _Status tracked from the sheet's **Status** column (col H) only. The separate "Status for G-ABLE"
> column (col R) is **not** used here, except to flag UAT **Reopen**._
>
> **Done-bucket rule:** **`Retest UAT` ≡ `Fixed` ≡ `Close` ≡ "done (dev-done)".** Only 🔴 Open /
> 🔎 Investigate / 🟡 Inprogress Fix / 🔵 Retest are genuinely outstanding.

> **The 3-pillar set:**
> - **Pillar 1 — All Tasks (this file):** every issue, grouped by layer, verbatim text + root cause + status.
> - **Pillar 2 — [UAT-<FEStack>-Plan-<Module>.md](UAT-<FEStack>-Plan-<Module>.md):** Before/After per frontend issue.
> - **Pillar 3 — [UAT-<BEStack>-Plan-<Module>.md](UAT-<BEStack>-Plan-<Module>.md):** Before/After per backend (.NET / SQL) issue.

## Sync log
> **Sync `(<prev>)` → `(<now>)` (delta, <date> — by Status column; file `(<now>)` is source of truth):**
> <only what changed: status flips grouped by transition, any NEW issues, new counts>.
> <append a NEW blockquote on each re-run; never rewrite earlier ones — this is the running history>

## Summary

| Status (sheet col H) | Count | Bucket |
|--------|-------|--------|
| 🟢 Close | <n> | ✅ done |
| 🔵 Retest UAT | <n> | ✅ done (≡ Fixed) |
| 🟢 Fixed | <n> | ✅ done |
| 🔴 Open | <n> | ⬜ remaining |
| 🔎 Investigate | <n> | ⬜ remaining |
| 🟡 Inprogress Fix | <n> | ⬜ remaining |
| 🔵 Retest | <n> | ⬜ remaining |
| **Total** | **<N>** | **Done <d> · Remaining <r>** |

| Layer | Count |
|----------|-------|
| FRONTEND | <n> |
| BACKEND | <n> |
| FULLSTACK | <n> |

**Owner legend:** 🅰️ Angular (frontend) · 🔷 .NET (backend) · 🗄️ SQL stored proc/DB · ❓ Needs clarification

### ✅ Tasks Left — **<r> remaining of <N>** (<d> done: `Retest UAT` ≡ `Fixed` ≡ `Close`)

> Only genuinely-outstanding items (🔴 Open / 🔎 Investigate / 🟡 Inprogress / 🔵 Retest), grouped by owner.

**🔷 .NET / C# — actionable now (<n>)**
- [ ] **BMS<id>** *(<status>)* — <one-line what + where>

**🗄️ SQL — DBA hand-off (<n>)**
- [ ] **BMS<id>** *(<status>)* — <one-line>

**🔎 Investigate / spec-gated — don't code blind (<n>)**
- [ ] **BMS<id>** — <why blocked>

**🅰️ Angular — remaining (<n>)**
- [ ] **BMS<id>** *(<status>)* — <one-line>

> **Next safest cluster:** <recommended order, lowest-risk grounded fixes first; defer repro/sign-off items>.

### Triage (quick scan — folded in from the retired task.md)

> Owner + can-fix + one-line root cause. Full per-issue detail (verbatim text, G-ABLE notes, checkboxes) is in the layer sections below — those are authoritative.

| BMS | Status | Can Fix? | Owner | Root cause (one line) |
|-----|--------|----------|-------|------------------------|
| BMS<id> | <status> | ✅/❓ | 🅰️/🔷/🗄️ | <one-line root cause> |

---
## Frontend — UI / Design / <FEStack>
*<n> issues*

### <emoji> BMS<id> | <Priority> | <Type> | Dev: <name> | <Status>
> Fix: <date> | <sync note, e.g. "Sync (9): Fixed → Retest UAT (≡ done; G-ABLE col R: Close)">

**Issue:**
- <original issue text, VERBATIM — keep source language, e.g. Thai>

**Remark (UAT):**
- <reviewer remark if present>

**G-ABLE Note:** <root cause + what shipped, in English; cite file:line as clickable links; link to the relevant plan file/section>

- [ ] Fix implemented
- [ ] Unit/integration test written
- [ ] UAT retest passed

---
## Backend — Logic / API / SQL
*<n> issues*

<same per-issue shape>

---
## Fullstack — Both layers
*<n> issues*

<same per-issue shape; fullstack issues listed once here, with a note that the FE half lives in the
 FE plan and the BE half in the BE plan>
````

Rules:
- Every tracked issue appears **exactly once** in a layer section (Frontend / Backend / Fullstack),
  **plus** one Triage row **plus** one Summary count.
- "Root cause (one line)" in the Triage table must match the per-issue G-ABLE Note's longer explanation.
- Original issue text + UAT remark stay **verbatim in source language** (e.g. Thai); English analysis
  goes only in **G-ABLE Note**.
- The header line MUST carry the full Status-column breakdown and which Excel export `(NN)` it synced from.
- Mark `*(repro)*` issues needing live reproduction and `*(new)*` issues new to this sync.
- Status is taken from the sheet's Status column, bucketed by the done-bucket rule above.

---

## Pillars 2 & 3 — By-stack Execution Plans (`UAT-<Stack>-Plan-<Module>.md`)

**One template, parameterized by the detected stack.** The FE plan uses `<FEStack>`/`ng build`;
the BE plan uses `<BEStack>`/`dotnet build` and includes SQL stored-proc changes when detected.
A fullstack issue appears in both plans, each showing only its own layer's Before/After.

This is where **priority/risk phasing lives** — phase issues lowest-risk first. (Pillar 1 stays
status-organized; the plans are risk-organized.)

````markdown
# UAT <Stack> Plan — <Module> (<layer>)

**Stack:** <Stack> <ver> · <arch/libs> · `<layer path>/` · branch `<branch>`
**Scope:** <which statuses, e.g. OPEN backend issues>. Phase 1 = grounded & proposed now; rest phased by risk.
Companion: [UAT-<otherStack>-Plan-<Module>.md](UAT-<otherStack>-Plan-<Module>.md) · board [<Module>-All-Tasks.md](<Module>-All-Tasks.md)

## Summary
| Status | Count |   | Phase | Theme | Count |
|--------|-------|---|-------|-------|-------|
| 🟢 Fixed | <n> |  | 1 | Grounded, low-risk (wording/popup/casing) | <n> |
| 🟡 Partial | <n> | | 2 | Code-fixable, needs care (shared files / logic guard) | <n> |
| 🔴 Open | <n> |   | 3 | Needs business / master-data / SAP input | <n> |
| **Total** | **<n>** | | 🗄️ | DB-only (`M_EMAIL_CONFIG`, SP) | <n> |

## Suggested Next Session
<short recommended order: lowest-risk grounded fixes first, defer items needing live reproduction / sign-off>

## Phase 0 — Setup (once)
- [ ] `cd <layer path>` · `<install>` · `<build>` (clean baseline) · branch `fix/...`

## Phase 1 — <Theme> (proposed now, grounded)
### <emoji> BMS<id> — <short title>
> Owner: 🅰️ Angular | 🔷 .NET | 🗄️ SQL · [`file.ext:<line>`](<path>#L<line>)
> <one-line root note>

**Before** (`:<line>`)
```csharp
oldCode(); // ← BUG: <why wrong>
```
**After** (`:<line>`)
```csharp
newCode(); // ← FIXED BMS<id>
```
> <dependency / caveat note if any>
☐ Fix · ☐ Test (<what the test asserts>) · ☐ UAT retest

## Phase 2 — code-fixable, needs care
- **BMS<id>** <one-line + file:line> …

## Phase 3 — needs business / master-data / SAP input
- **BMS<id>** <one-line> …

## 🗄️ DB-only (`M_EMAIL_CONFIG` / stored procs)
- **BMS<id>** <one-line of the data/SP change> …

## Final Verify
- [ ] Build (zero errors) + lint · walk each item Before → After against the UAT description
- [ ] Re-confirm *(approximate)* snippets against the live file before committing
````

- Group issues into phases by **risk and shared files** — lowest-risk grounded fixes (wording, casing,
  popup text) in Phase 1; shared-component / logic-guard changes in Phase 2; business/master-data/SAP
  sign-off items in Phase 3; pure DB/stored-proc edits in their own 🗄️ section.
- FE plan phases lean Layout/CSS/validation-styling; BE plan phases lean SQL/data/handler logic.
- Use the stack's real build command in Phase 0 / Final Verify (`ng build` vs `dotnet build`).
- Single-line `☐ Fix · ☐ Test · ☐ UAT retest` is fine inside dense plans; the multi-line checkbox form
  is reserved for Pillar 1.

---

## Cross-pillar consistency (check before finishing)

- Same set of issue IDs across all files; each issue's **status is identical** everywhere and matches
  the sheet's Status column.
- A fix → update the Before/After block (Fixed note) → tick the box in the plan → tick the box in
  All-Tasks → flip the All-Tasks status + Summary counts + the Sync-log delta.
- Fullstack issue = once in All-Tasks (Fullstack section) + a FE half in the FE plan + a BE half in
  the BE plan. Don't duplicate the same Before/After across both plans.
- Every Before block in a plan was read from the real tree, or is explicitly `*(approximate)*`.
- Sync-discrepancy issues (sheet says Open but code is fixed, or col R = Reopen) are flagged in all files.
