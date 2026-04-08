# 4. Skills: knowledge on demand

Skills are bundles of instructions and resources that the agent loads **only when relevant**. They solve a problem `AGENTS.md` can't: how to give the agent deep domain knowledge without bloating its context on every turn.

## What you'll learn

- What a skill is and how it differs from `AGENTS.md` content.
- How skills get discovered and activated.
- When to create a skill vs when to add a line to `AGENTS.md`.

## Outline

1. The bloat problem: why you can't just put everything in `AGENTS.md`.
2. Skills as lazy-loaded knowledge: only present in context when relevant.
3. Anatomy of a skill: trigger description, body, optional resources.
4. Discovery: how the agent decides to load a skill.
5. Examples: a "writing migrations" skill, a "reviewing PRs" skill, a "Spring Boot conventions" skill.
6. Decision rule: always-true → `AGENTS.md`. Sometimes-true → skill.

## Key takeaway

Skills are how you scale knowledge without scaling context. If a piece of guidance only matters for 10% of tasks, it doesn't belong in `AGENTS.md`.
