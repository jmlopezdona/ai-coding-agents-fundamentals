# 11. Memory and persistence across sessions

Agents have several ways to "remember" things: the live conversation, persistent memory files, plans, task lists, and the repository itself. Each of these has a different lifespan, a different audience, and a different failure mode. Treating them as interchangeable is one of the most common early mistakes — and one of the biggest sources of silent quality loss as a team scales.

## What you'll learn

- The four persistence layers, plus the broader fifth layer of project knowledge artifacts.
- Why dumping everything into "memory" is a bad idea.
- A decision tree for where a given piece of information belongs.
- How to write a memory file you won't regret in three months.

## The four persistence layers

Think of agent memory as a stack of layers, each with a shorter lifespan than the one below it.

### Layer 1: In-session context (the context window)

This is everything the model currently "sees" — the system prompt, tool definitions, conversation history, files it has read, and tool outputs. It lives only for the current task. When the session ends (or the window is compacted), it's gone.

Use it for: the problem you are solving right now, intermediate reasoning, file contents the agent just opened, draft plans.

!!! warning "Context is not free"
    Every token you stuff into context costs latency, money, and attention. A 200k window does not mean you should fill it. Agents reason worse in the last 20% of a full window than in the first 20%.

### Layer 2: Plans and task lists (session memory)

Tools like Claude Code's todo list, Cursor's plan panel, or Aider's chat history give the agent a scratchpad that survives within a session but not beyond it. This is where multi-step work lives: "I'm on step 3 of 7, step 2 failed for reason X, here's the retry."

Use it for: the current multi-step task, in-flight decisions, things you want the agent to come back to before the session ends.

### Layer 3: Project memory (AGENTS.md / CLAUDE.md)

This is the first truly durable layer. Files like `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, or `.aider.conf.yml` live in the repo and are loaded automatically at the start of every session. They describe *the project*: how to run tests, the architecture style, naming conventions, which directories are off-limits, how to deploy.

Because they live in the repo, they are shared with the whole team and versioned with the code. This is where project-level knowledge belongs — not in any individual's private memory.

### Layer 4: Persistent user memory

Most coding agents also offer a private, per-user memory — Claude Code's `~/.claude/CLAUDE.md`, Cursor's user rules, Codex's user instructions. This layer survives across projects and sessions. It's for facts about *you*: "I prefer pnpm over npm", "I work on macOS with fish shell", "don't suggest emojis in commit messages".

!!! tip "The golden rule"
    If a teammate would also benefit from knowing it, it belongs in the repo (layer 3), not in your personal memory (layer 4).

### Layer 5: Project knowledge artifacts (on-demand)

The four layers above answer "what does the agent always know?". There's a much wider body of knowledge that belongs to the project but should *not* live in `AGENTS.md` because it's too big and too rarely needed every turn. This is the **on-demand knowledge layer**: long-form artifacts the agent reads only when relevant to the current task.

Typical artifacts:

- **Functional documentation** — specs, requirements, closed user stories.
- **Architecture documentation** — C4 diagrams, domain models, service contracts.
- **ADRs** (Architecture Decision Records) — the historical *why* behind decisions.
- **Execution plans** — roadmaps, migration plans, in-flight phases.
- **Operational runbooks** — what to do when X fails, how to deploy Y.
- **Post-mortems and incident reports** — historical lessons.
- **Glossaries and domain models** — the business vocabulary.
- **Internal API documentation** — OpenAPI specs, event schemas.

Why this matters: when the agent is implementing "add an endpoint for X", the technical context lives in the code, but the context of *why* and *how it fits* lives in these documents. Without them, the agent makes locally reasonable decisions that are globally inconsistent with the system.

This is different from the layers above:

- **Not `AGENTS.md`** — that file is short and always-on. These artifacts are long and on-demand.
- **Not a skill** — skills are procedural ("how to do X"). These are descriptive ("what X is and why").

There are three good ways to expose them to the agent:

1. **Versioned files in the repo** (`docs/`, `adr/`) — the agent reads them with its filesystem tool. Best when the artifacts belong to the project and should travel with the code.
2. **Skills that reference them** as supporting files via progressive disclosure (chapter 5) — the skill's `SKILL.md` is loaded always, the long reference file only when the skill activates.
3. **An MCP server** over the corporate wiki/CMS (Confluence, Notion, SharePoint — chapter 8) — when the artifacts live outside the repo and are shared across projects.

The decision: **in repo when project-scoped and versionable; in MCP when corporate-wide and shared.**

The key piece — and the part most teams miss — is that these artifacts have to be **discoverable from somewhere the agent already reads**. A short pointer in `AGENTS.md` is usually enough:

```markdown
## Project knowledge

- Architecture overview: `docs/architecture/overview.md`
- ADRs: `docs/adr/` (read the index first)
- Payments domain — read `docs/adr/0007-payments.md` before touching `app/payments/`
- Runbooks: `docs/runbooks/`
```

!!! warning "Two anti-patterns to avoid"
    **Anti-pattern 1: dump it all into `AGENTS.md`.** Pasting the architecture doc inline blows up the context window on every turn. Point at it, don't paste it.

    **Anti-pattern 2: have the artifacts but never link to them.** If `AGENTS.md` doesn't tell the agent that `docs/adr/` exists, the agent won't find it. The artifact might as well not exist.

## A decision tree

When you find yourself wanting the agent to "remember" something, ask:

1. **Is it relevant only to the current task?** → Keep it in context. Do nothing.
2. **Is it relevant for the rest of this session?** → Put it in the plan or todo list.
3. **Is it a short, always-true fact about the project?** → `AGENTS.md` in the repo.
4. **Is it long-form project knowledge (architecture, ADRs, runbooks)?** → Layer 5 — versioned doc in the repo (or MCP if it lives in the wiki), with a pointer from `AGENTS.md`.
5. **Is it a personal preference about how *you* work?** → User-level memory file.
6. **Is it a one-off observation you'll never need again?** → Don't write it anywhere.

## An example memory file

Here is a small, well-scoped `AGENTS.md` — the kind that actually helps:

```markdown
# Project: billing-api

## Stack
- Python 3.12, FastAPI, SQLAlchemy 2.x, Postgres 16.
- Tests: pytest + pytest-asyncio. Run with `make test`.
- Lint/format: `make lint` (ruff + mypy strict).

## Conventions
- All new endpoints go under `app/api/v1/`.
- Database models live in `app/models/`. Never import them from `app/api/`
  directly — always go through a repository in `app/repositories/`.
- Money is always `Decimal`, never `float`. Currency is a separate field.

## Definition of done
- `make lint test` passes.
- New endpoints have a request/response schema and an integration test.
- No changes to `alembic/versions/` without an accompanying migration note.

## Off-limits
- `infra/` and `.github/` — ask before touching.
```

Notice what is *not* in there: no history of past bugs, no changelog, no personal opinions, no 30-line explanation of REST. It is short, scannable, and every line earns its place.

## The anti-pattern: memory as a notebook

The most common failure mode is using memory as a diary. Every time something goes slightly wrong, a new line gets appended: "Remember that on 2026-03-14 the test for X failed because Y." After a month, the memory file is 2000 lines of stale anecdotes that pushes out the instructions the agent actually needs.

Memory is not a log. It is a *curated* set of non-obvious facts. If a note isn't going to matter next week, it doesn't belong there. Prune ruthlessly — deleting lines from a memory file is almost always a net improvement.

!!! success "Key takeaway"
    Each persistence layer has a job. The repo (`AGENTS.md`) is the only layer your teammates can see — anything important to the *team* belongs there, not in any one developer's private memory. When in doubt, prefer the shorter memory and the longer repo.
