# G-ABLE Skills

Shared [Agent Skills](https://github.com/vercel-labs/skills) for the G-ABLE team — installable into
Claude Code (and Cursor, Codex, OpenCode, etc.) with one command.

## Install

> GitHub path: `fngage00789/gable-skills`

```bash
# install everything in this repo
npx skills add fngage00789/gable-skills

# or pick specific skills
npx skills add fngage00789/gable-skills --skill excel-issue-fixer

# list what's available without installing
npx skills add fngage00789/gable-skills --list

# target a specific agent (default: auto-detect installed agents)
npx skills add fngage00789/gable-skills -a claude-code
```

The CLI fetches each `SKILL.md` from GitHub and installs it into your agent's skills directory
(`.claude/skills/` for Claude Code). No npm publish needed — GitHub is the registry.

Manage installed skills:

```bash
npx skills list      # show installed
npx skills update    # pull latest
npx skills find       # search
```

## Skills in this repo

| Skill | What it does |
|-------|--------------|
| [`excel-issue-fixer`](skills/excel-issue-fixer/) | Read an Excel UAT/issue sheet, classify FE/BE/fullstack, auto-detect the stack, and generate the **3-pillar** output (All-Tasks board + a Before/After execution plan per detected stack), then implement and keep all three in sync. |

## Repo layout

```
skills/
  excel-issue-fixer/
    SKILL.md            # frontmatter: name + description (how the agent decides to trigger)
    references/         # loaded on demand by the skill
      classification-rules.md
      excel-parsing.md
      three-pillar-output.md
```

`npx skills` supports a flat layout (`skills/<name>/SKILL.md`) and a catalog layout
(`skills/<category>/<name>/SKILL.md`). Add new skills as sibling folders under `skills/`.

## Adding a new skill

1. Create `skills/<your-skill>/SKILL.md` with YAML frontmatter (`name`, `description`).
2. Put any supporting docs/scripts in `skills/<your-skill>/references/` (or `scripts/`).
3. Commit and push — teammates pick it up with `npx skills update`.
