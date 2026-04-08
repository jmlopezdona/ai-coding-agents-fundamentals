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

## Progressive disclosure: referenced files on demand

A skill isn't just a single `SKILL.md`. It's a directory, and that directory can bundle additional material: longer reference docs, example files, templates, golden snippets, a cheatsheet for an obscure API. The trick is that none of it is loaded up front. The agent reads the `SKILL.md` body, and *only if* it decides a particular referenced file is relevant to the current task does it open it.

This is progressive disclosure: the skill advertises what's available, and the agent pulls the deep material on demand. The footprint stays small — a description, a short body, a list of pointers — until a task actually needs the 400-line migration cookbook or the legacy schema dump.

!!! note "Why this matters"
    Contrast with stuffing the cookbook into `AGENTS.md`: you'd pay the token cost on *every* turn, for every task, forever. With a referenced file inside a skill, you pay it only when the skill triggers *and* the referenced file is relevant. Same knowledge, a fraction of the cost.

## Bundled executable scripts

A skill can also ship with scripts — `scripts/validate.sh`, `generate.py`, `scaffold.ts`, whatever — that the agent runs directly in the local environment when the skill applies. This is the in-process alternative to MCP: no server, no protocol, no lifecycle. Just code that lives next to the instructions and that the agent invokes when it's relevant.

Scripts are a great fit for deterministic work: validating a generated migration, scaffolding a new module from a template, running a codemod, transforming a payload, checking that a file matches a schema. The skill body tells the agent *when* to run the script and *how to interpret* the output.

!!! tip "Skills with scripts vs MCP"
    Skills with bundled scripts are simpler to ship — they're just files in the repo, versioned alongside the code. They run locally, in the agent's own environment, with the same credentials and filesystem. MCP servers are heavier (a running process, a protocol, auth) but they can wrap *remote* systems and be shared across projects and clients. Rule of thumb: if the behavior is local and deterministic, a script in a skill is the lighter path. If it needs to call a remote service or be reused by several agents, reach for MCP.

### Sample SKILL.md with references and a script

```markdown
---
name: writing-flyway-migrations
description: >
  Use when creating or reviewing Flyway migrations under
  src/main/resources/db/migration, or on mentions of "migration",
  "ALTER TABLE", or "Flyway".
version: 1.3.0
---

# Writing Flyway migrations

Follow the two-step rule for destructive changes. Full cookbook in
`reference/destructive-changes.md` — open it when the change touches
an existing column or index.

For the file name and header, use the template in
`templates/migration.sql.tmpl`.

## Validation

After writing the migration, run:

    scripts/validate-migration.sh path/to/VXXXX__name.sql

The script checks naming, idempotency, and that `CONCURRENTLY` is
used on indexed tables > 100k rows. Fix any reported issue before
handing the change back to the user.
```

The directory on disk looks like:

```
writing-flyway-migrations/
├── SKILL.md
├── reference/
│   └── destructive-changes.md
├── templates/
│   └── migration.sql.tmpl
└── scripts/
    └── validate-migration.sh
```

Only `SKILL.md` is loaded when the skill triggers. The reference doc and the template are opened on demand; the script is executed on demand.

## Discovery: auto vs explicit

There are two ways a skill ends up in context, and they're not mutually exclusive.

- **Auto-discovery.** The agent runtime scans the available skills each turn, reads just the names and descriptions, and decides which ones apply to the current task. This is what the matcher section above describes. It's powerful but it depends entirely on the quality of the agent's implementation — and on how precise your trigger descriptions are.
- **Explicit reference.** An agent or subagent definition lists the skills it should always have access to. For example, a dedicated "migrations" subagent can be wired to always load the `writing-flyway-migrations` skill, regardless of what the matcher thinks. More predictable, less magical, and useful when you want guaranteed behavior for a specific role.

!!! tip "Use both"
    Auto-discovery is great for the long tail of skills that apply occasionally across many tasks. Explicit reference is great for the few skills that *define* what a subagent is for. A well-configured setup uses both: auto-discovery for breadth, explicit wiring for the critical path.

## Claude Code: skills as user-invocable units

Claude Code adds a twist worth calling out: skills can be invoked not only by the agent (auto-discovered or explicitly wired) but also **by the user directly**, the same way you'd call a slash command. The user types the skill's name, the skill loads, and the agent runs with it.

This blurs the line between "skill" and "command". In practice, skills are progressively replacing the older slash-command concept in Claude Code, because a skill bundles everything a command used to do *and more*: instructions, referenced files, executable scripts, and richer metadata — all in one versioned package.

!!! note "Vendor-specific"
    This user-invocable behavior is a Claude Code choice. Other agents may expose skills only to the runtime, or keep slash commands as a separate concept entirely. Check your agent's documentation before assuming the same ergonomics carry over.

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
