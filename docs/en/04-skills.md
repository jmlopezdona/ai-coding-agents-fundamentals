# 4. Skills: knowledge on demand

Skills are bundles of instructions and resources that the agent loads **only when relevant**. They solve a problem `AGENTS.md` can't: how to give the agent deep domain knowledge without bloating its context on every turn.

## What you'll learn

- What a skill is and how it differs from `AGENTS.md` content.
- How skills get discovered and activated.
- The anatomy of a skill file, including a real frontmatter example.
- A simple decision rule for *skill vs `AGENTS.md`*.

## The bloat problem

`AGENTS.md` (or `CLAUDE.md`, `.cursorrules`, `AGENT.md` — the name varies, the idea doesn't) is loaded into context on every turn. That's exactly what you want for things that are *always* true: the language, the package manager, the build command, the team's non-negotiables.

But what about the 200 lines explaining how to write a Flyway migration? The 80 lines on your custom MapStruct conventions? The runbook for deploying the legacy worker? If you put all of it in `AGENTS.md`, every turn — even "rename this variable" — pays the token cost. Worse, the agent's attention dilutes: with too much always-on guidance, none of it is salient.

!!! note "Context is a budget, not a bucket"
    Every token you load is a token that competes with the actual task. The goal isn't to load *more*; it's to load *the right things*.

## Skills as lazy-loaded knowledge

A skill is a self-contained chunk of guidance that sits on disk and is *only* pulled into context when the current task matches its trigger description. Think of it as `import` for prompts: the module exists, but it's only loaded when needed.

This is the model used by Claude Code skills, by Cursor's rule files with `globs:` triggers, by Codex's contextual instructions, and by community tools like Aider's conventions split across files.

## Anatomy of a skill

A typical skill is a directory with a `SKILL.md` (or equivalent) file at the root. The file has two parts: **frontmatter** that tells the agent *when* to load it, and a **body** that tells the agent *what to do* once loaded.

### Sample SKILL.md frontmatter

```markdown
---
name: writing-flyway-migrations
description: >
  Use when creating, editing, or reviewing Flyway database migrations
  for the Postgres-backed services in this repo. Triggers on changes
  under src/main/resources/db/migration or mentions of "migration",
  "schema change", "ALTER TABLE", or "Flyway".
version: 1.2.0
---

# Writing Flyway migrations

## Naming
- File pattern: `V{yyyyMMddHHmm}__{snake_case_description}.sql`
- Never edit a migration that has been merged to `main`.

## Safety rules
- Always wrap destructive changes (DROP, ALTER COLUMN TYPE) in a
  two-step migration: add the new shape, backfill, then remove the old.
- Never use `CREATE INDEX` without `CONCURRENTLY` on tables > 100k rows.

## Required checklist
1. Migration runs cleanly on an empty DB.
2. Migration runs cleanly on a copy of production schema.
3. Rollback strategy documented in the PR description.

## Resources
- See `examples/V202401151200__add_user_email_index.sql` for a reference.
```

The `description` field is the most important line in the file. It's what the agent reads to decide whether to load the skill. Vague descriptions ("helps with databases") never trigger; precise ones ("use when editing files under `db/migration` or when the user mentions Flyway") trigger reliably.

## Discovery: how skills get activated

Most agents do something like this on each turn:

1. Read the list of available skills (just the names and descriptions, cheap).
2. Compare them against the current task — file paths touched, keywords in the user's message, recent tool output.
3. Load the body of any skill whose description clearly matches.

!!! tip "Write descriptions for the matcher, not for humans"
    Include the exact filenames, glob patterns, library names, and verbs your team uses. "Reviewing PRs" is a worse trigger than "Use when running `/review-pr`, when the user pastes a GitHub PR URL, or when asked to review a diff."

## Examples that pay off

- **`writing-migrations`** — schema rules, naming, two-step destructive changes.
- **`reviewing-prs`** — your review rubric, the severities, the tone.
- **`spring-boot-conventions`** — package layout, configuration patterns, testing style.
- **`incident-response`** — runbook for the on-call agent: where logs live, who to page, what *not* to touch.

Each of these would be noise in `AGENTS.md` for 90% of tasks. As skills, they're invisible until they're useful.

## The decision rule

!!! success "Skill vs AGENTS.md"
    - **Always true?** It belongs in `AGENTS.md`. (Build command, language, lint rules.)
    - **Sometimes true?** It belongs in a skill. (Migration rules, deployment runbook, niche refactors.)
    - **Only true once?** It belongs in the prompt. Don't over-engineer.

## Failure modes

!!! warning "Skills can rot too"
    - **Never-triggering skills.** The description is too vague — the agent can't tell when to load it. Fix the trigger, not the body.
    - **Always-triggering skills.** The description is too broad and the skill is loaded on every turn. That's just `AGENTS.md` with extra steps.
    - **Stale skills.** The library upgraded, the conventions changed, no one updated the file. Treat skills like code: review and version them.

## Key takeaway

!!! success "Key takeaway"
    Skills are how you scale knowledge without scaling context. If a piece of guidance only matters for 10% of tasks, it doesn't belong in `AGENTS.md` — it belongs in a skill with a precise trigger description.
