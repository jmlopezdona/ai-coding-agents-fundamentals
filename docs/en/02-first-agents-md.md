# 2. Your first AGENTS.md

`AGENTS.md` (or `CLAUDE.md`, `.cursorrules`, etc.) is the file the agent reads before every task. It's your most direct lever for shaping behavior. The most common mistake is to dump the entire project documentation into it. That's not what it's for.

## What you'll learn

- What `AGENTS.md` is for and what it isn't.
- The principle: **instructions, not encyclopedia**.
- A minimal template you can adapt.

## The same file, many names

Every major coding agent has a "project instructions" file that gets injected into context on every turn:

| Tool | File |
|------|------|
| OpenAI Codex CLI, Aider, generic | `AGENTS.md` |
| Claude Code | `CLAUDE.md` (also reads `AGENTS.md`) |
| Cursor | `.cursorrules` or `.cursor/rules/*.mdc` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Continue | `.continuerules` |

The filename differs; the role is identical: a short, version-controlled file that tells the agent how *this specific repo* wants to be worked on. In this guide we'll call it `AGENTS.md` generically.

!!! note "It's loaded every turn"
    Unlike the README, this file is *always* in context. Every line costs tokens on every turn. That's why brevity matters — and why it's powerful.

## What to put in

Put in things the agent **cannot infer from the code itself** and that **change its behavior**. Good candidates:

- **How to run things**: the exact command for tests, lint, typecheck, build. Not "we use pytest" — `pytest -xvs tests/`.
- **Source of truth**: which directory is generated, which is hand-written, where types are defined.
- **Conventions** the linter doesn't catch: "use `Result<T, E>` not exceptions for domain errors", "all DB access goes through the repository layer".
- **Things to avoid**: "do not edit files under `vendor/`", "do not bump dependencies without asking", "do not add new top-level dependencies".
- **Local quirks**: "the integration tests need Docker running", "node_modules is at the repo root, not per-package".

## What to leave out

- Architectural deep-dives. The agent will read the code if it needs to.
- History and rationale. Useful for humans, noise for agents.
- Anything obvious from the repo structure (`src/` contains source — yes, thank you).
- The full list of every endpoint, every model, every table. Let search tools do that work.
- Marketing copy from the README.

!!! warning "Encyclopedias get ignored"
    A 2000-line `AGENTS.md` doesn't make the agent smarter. It pushes useful signal out of the context window and trains you not to read the file when something needs to change. Keep it under a page if you can.

## The "instructions, not encyclopedia" principle

Read every line of your `AGENTS.md` and ask: *"if I delete this line, will the agent behave worse on some real task?"* If the honest answer is no, delete it. Every surviving line should be a behavior change waiting to happen.

## A worked example

Here's a realistic `AGENTS.md` for a small Python web service. It's about 40 lines, and every line is there for a reason.

```markdown
# AGENTS.md

This is `billing-service`, a FastAPI app that issues invoices.
Python 3.11, managed with `uv`.

## Commands

- Install deps:        `uv sync`
- Run tests:           `uv run pytest -xvs`
- Run a single test:   `uv run pytest -xvs tests/test_invoices.py::test_total`
- Lint + format:       `uv run ruff check --fix && uv run ruff format`
- Type check:          `uv run mypy src/`
- Run locally:         `uv run uvicorn billing.app:app --reload`

Always run lint, typecheck, and the tests before declaring a task done.

## Layout

- `src/billing/`        — application code (hand-written, source of truth)
- `src/billing/api/`    — FastAPI routers, thin; no business logic here
- `src/billing/domain/` — pure domain logic, no I/O, fully unit-tested
- `src/billing/db/`     — SQLAlchemy models and repositories
- `tests/`              — pytest, mirrors the `src/billing/` layout
- `migrations/`         — Alembic, generated; do not hand-edit

## Conventions

- Money is `decimal.Decimal`, never `float`. Round half-even.
- Domain errors are explicit exception classes in `billing.domain.errors`,
  not generic `ValueError`.
- All DB access goes through a repository in `billing/db/repositories/`.
  Routers and services never import `sqlalchemy` directly.
- Public functions have type hints and a one-line docstring.

## Things not to do

- Do not add new top-level dependencies without asking.
- Do not edit files under `migrations/versions/` by hand;
  generate a new migration with `alembic revision --autogenerate`.
- Do not catch `Exception` broadly; catch the specific domain error.
- Do not commit changes to `.env` or anything in `secrets/`.
```

Notice what's *not* in there: no description of what an invoice is, no list of endpoints, no architecture diagram, no history. The agent can find all of that by reading the code. What it cannot find by reading the code is *the команды you actually use*, *which directories are generated*, and *what to avoid*.

## How to evolve it

Your `AGENTS.md` is a living file, but it should grow slowly and on evidence.

!!! tip "The two-strikes rule"
    Add a line to `AGENTS.md` the second time the agent gets the same thing wrong. Once might be a fluke. Twice means the information is missing from context, and a single line can fix it forever.

Conversely, prune. If a line was added for an old convention and that convention has changed, delete it — stale instructions are worse than no instructions, because the agent will follow them confidently.

## Key takeaway

!!! tip "Key takeaway"
    A short, opinionated `AGENTS.md` that gets read every turn beats a 2000-line one that gets ignored. Start with 30 lines. Earn each addition.
