# 2. Your first AGENTS.md

`AGENTS.md` (or `CLAUDE.md`, `.cursorrules`, etc.) is the file the agent reads before every task. It's your most direct lever for shaping behavior. The most common mistake is to dump the entire project documentation into it. That's not what it's for.

## What you'll learn

- What `AGENTS.md` is for and what it isn't.
- The principle: **instructions, not encyclopedia**.
- A minimal template you can adapt.

## Outline

1. The file across vendors: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md`. Same idea, different filenames.
2. What to put in: how to run tests, where the source of truth lives, conventions the agent can't infer from the code, things to avoid.
3. What to leave out: architectural deep-dives, history, anything already obvious from the repo structure.
4. The "instructions, not encyclopedia" principle — every line should change behavior.
5. A minimal worked example for a typical Node.js / Python / Java repo.
6. How to evolve the file: add a line each time the agent gets something wrong twice.

## Key takeaway

A short, opinionated `AGENTS.md` that gets read every turn beats a 2000-line one that gets ignored. Start with 30 lines. Earn each addition.
