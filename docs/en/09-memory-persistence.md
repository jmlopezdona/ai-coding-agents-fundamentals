# 9. Memory and persistence across sessions

Agents have several ways to "remember" things: in-conversation context, persistent memory files, plans, task lists, and the repo itself. Each has a different lifespan and purpose. Mixing them up is one of the most common early mistakes.

## What you'll learn

- The four persistence layers and what each is for.
- Why dumping everything into "memory" is a bad idea.
- A decision tree: where does this piece of information belong?

## Outline

1. Layer 1: **conversation context** — lives for the current task.
2. Layer 2: **plans and tasks** — live for the current session.
3. Layer 3: **memory** — survives across sessions, but should be rare and curated.
4. Layer 4: **the repo** — `AGENTS.md`, skills, commands. The only truly durable layer.
5. Decision tree: ephemeral → context; this session → plan/tasks; future sessions about *you* → memory; future sessions about *the project* → repo.
6. Anti-pattern: using memory as a notebook. Memory is for non-obvious facts about the user and the project, not a log.

## Key takeaway

Each persistence layer has a job. The repo is the only one your teammates can see — anything important to the *team* belongs there, not in any one developer's memory.
