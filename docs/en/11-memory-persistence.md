# 11. Memory and persistence across sessions

Agents have several ways to "remember" things: the live conversation, persistent memory files, plans, task lists, and the repository itself. Each of these has a different lifespan, a different audience, and a different failure mode. Treating them as interchangeable is one of the most common early mistakes — and one of the biggest sources of silent quality loss as a team scales.

## What you'll learn

- The four persistence layers and what each is for.
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

## A decision tree

When you find yourself wanting the agent to "remember" something, ask:

1. **Is it relevant only to the current task?** → Keep it in context. Do nothing.
2. **Is it relevant for the rest of this session?** → Put it in the plan or todo list.
3. **Is it a fact about the project that every teammate should see?** → `AGENTS.md` in the repo.
4. **Is it a personal preference about how *you* work?** → User-level memory file.
5. **Is it a one-off observation you'll never need again?** → Don't write it anywhere.

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
