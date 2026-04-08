# 8. Permissions, sandbox, and execution modes

Every agent product has knobs for "how much can it do without asking?" Get this wrong in either direction and you'll suffer: too restrictive and the agent is useless, too permissive and you'll regret it the first time it runs `rm -rf` or pushes to `main`.

## What you'll learn

- The execution modes most agents offer (plan, ask, auto-accept).
- How to choose autonomy per tool, not per session.
- A safe default for a team starting out.

## Outline

1. The autonomy spectrum: plan-only → ask-each-tool → auto-accept-allowlist → full-auto.
2. Why "full-auto for everything" is a beginner trap.
3. Per-tool permissions: trust `Read` and `Grep`, gate `Bash` and `Write`.
4. Sandboxes: containers, worktrees, ephemeral environments.
5. Safe defaults for a team's first month.
6. When to relax permissions and how to do it deliberately, not by frustration.

## Key takeaway

Choose autonomy per tool, not per session. The cost of one prompt-confirmation is tiny; the cost of one unwanted `git push --force` is enormous.
