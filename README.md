# G-ABLE Skills

Shared [Agent Skills](https://github.com/vercel-labs/skills) for the G-ABLE team — installable into
Claude Code (and Cursor, Codex, OpenCode, etc.) with one command.

## Install

> GitHub path: `fngage00789/gable-dotnetfix-skills`

```bash
# install everything in this repo
npx skills add fngage00789/gable-dotnetfix-skills

# or pick specific skills
npx skills add fngage00789/gable-dotnetfix-skills --skill excel-issue-fixer
npx skills add fngage00789/gable-dotnetfix-skills --skill dotnet-developer

# list what's available without installing
npx skills add fngage00789/gable-dotnetfix-skills --list

# target a specific agent (default: auto-detect installed agents)
npx skills add fngage00789/gable-dotnetfix-skills -a claude-code
```

The CLI fetches each `SKILL.md` from GitHub and installs it into your agent's skills directory
(`.claude/skills/` for Claude Code). No npm publish needed — GitHub is the registry.

## Use a skill

After installing, restart Claude Code (or run `/skills`) and invoke a skill by name:

```text
/dotnet-developer        # senior .NET developer (.NET 8, ASP.NET Core, EF Core, …)
/excel-issue-fixer       # Excel UAT/issue sheet → 3-pillar plan + implementation
```

Skills also trigger automatically when your request matches their keywords — e.g. asking to
"write unit tests for this service" activates `/dotnet-developer`.

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
| [`dotnet-developer`](skills/dotnet-developer/) | Senior .NET developer — defaults to .NET 8, writes compilable code, and always follows a fix with tests. Routes across C#, ASP.NET Core, EF Core, Azure, Clean Architecture/CQRS, Blazor, DevOps, testing, security, and observability via on-demand reference docs. |

## Repo layout

```
skills/
  excel-issue-fixer/
    SKILL.md            # frontmatter: name + description (how the agent decides to trigger)
    references/         # loaded on demand by the skill
      classification-rules.md
      excel-parsing.md
      three-pillar-output.md
  dotnet-developer/
    SKILL.md            # router + rules + after-every-fix test-generation workflow (.NET 8)
    README.md           # install + usage for this skill
    references/         # loaded on demand by the skill
      csharp-patterns.md
      aspnetcore-patterns.md
      data-patterns.md
      azure-patterns.md
      architecture-patterns.md
      blazor-patterns.md
      devops-patterns.md
      security-patterns.md
      observability-patterns.md
      testing-patterns.md
```

`npx skills` supports a flat layout (`skills/<name>/SKILL.md`) and a catalog layout
(`skills/<category>/<name>/SKILL.md`). Add new skills as sibling folders under `skills/`.

## Adding a new skill

1. Create `skills/<your-skill>/SKILL.md` with YAML frontmatter (`name`, `description`).
2. Put any supporting docs/scripts in `skills/<your-skill>/references/` (or `scripts/`).
3. Commit and push — teammates pick it up with `npx skills update`.
