# 3. Rules and instructions: same problem, different formats

Every team eventually wants the agent to *just stop* doing something — stop inventing import paths, stop using `any`, stop reformatting files it wasn't asked to touch. Retyping "please don't do X" on every turn is not a solution. That's the problem rules and instructions solve: persistent guidance the agent carries into every task without you having to remind it.

## What you'll learn

- What "rules" and "instructions" actually are, and how they differ from a prompt.
- Why some tools give you a dedicated rules system and others fold it into `AGENTS.md`.
- How the main coding agents handle this, side by side.
- The difference between rules, `AGENTS.md`, and skills — and the trap of duplicating them.

## Rules, in one sentence

A rule is **always-on (or scope-on) guidance** that the agent sees without being asked. "Use `Result<T, E>` instead of throwing." "Never edit files under `generated/`." "All React components are function components with hooks." They're short, declarative, and they travel with the repo (or with your user profile) rather than living in a one-shot prompt.

Rules are *not* a prompt — a prompt is a single request. Rules are *not* a skill either — a skill is on-demand knowledge the agent loads when it thinks it's relevant. Rules are the background radiation: they're always there, shaping behavior whether the current task mentions them or not.

## Why a separate mechanism at all?

Here's the interesting part: some agents expose a dedicated rules system (Cursor, Windsurf, Cline, Continue), and others don't (Claude Code, Codex CLI). Both groups are solving the **same** underlying problem — giving the agent always-on guidance without the user retyping it — they just disagree on the shape of the solution.

The dedicated-rules camp argues: rules are a different kind of thing from project briefing. They deserve their own files, their own activation logic (by glob, by task type, always-on), and their own review workflow. Letting you write `react.mdc` that only applies to `*.tsx` files is genuinely useful.

The fold-it-into-AGENTS.md camp argues: every extra mechanism is another place to look when behavior goes wrong. One file, one source of truth, no "did I put this in rules or in CLAUDE.md?" confusion.

Both are defensible. Neither is wrong. But if you hop between tools, you need to know which model each one uses.

## How the main tools handle it

| Tool | File | Scope | Conditional? |
|------|------|-------|--------------|
| Cursor | `.cursor/rules/*.mdc` | Repo + user | Yes — `alwaysApply`, `globs`, `description` frontmatter |
| GitHub Copilot | `.github/copilot-instructions.md` + user settings | Repo + user | No — always applied |
| Windsurf | `.windsurfrules` + global rules | Repo + user | Limited |
| Cline | `.clinerules` file or `.clinerules/` directory | Repo | File-based activation |
| Continue | Rules in config + `.continuerules` | Repo + user | Partial |
| Aider | `CONVENTIONS.md` via `--read` | Repo (explicit load) | No — you choose when to load |
| Claude Code | `CLAUDE.md` / `AGENTS.md` | Repo + user | No — single always-on file |
| Codex CLI | `AGENTS.md` | Repo + user | No — single always-on file |

The column that matters most is the last one. **Conditional rules** — "only apply this when the agent is touching TypeScript" — are the main reason a dedicated system earns its keep. If you don't need that, `AGENTS.md` does the job.

## Rules vs AGENTS.md vs skills

These three mechanisms get confused constantly. The distinction is about **when the content is in context**:

| Mechanism | When it's loaded | Typical content |
|-----------|------------------|-----------------|
| Rules | Always, or on glob/scope match | Short do/don't statements, conventions |
| `AGENTS.md` | Always (every turn) | Project briefing, commands, conventions, guardrails |
| Skills | On demand, when the agent decides it's relevant | How-to knowledge, procedures, reference material |

- **Rules** are tight and declarative: "always", "never", "prefer".
- **`AGENTS.md`** is the project briefing: what this repo is, how to run it, what matters. Rules are often *a subset* of what lives in `AGENTS.md`.
- **Skills** are pull-not-push: the agent reaches for them when the task calls for them, and they stay out of context the rest of the time.

## The duplication trap

The most common failure mode is writing the same convention into all three places. "Use `Result<T, E>`" ends up in `.cursor/rules/errors.mdc`, in `AGENTS.md`, and in a `error-handling` skill. Now you have three copies to keep in sync, and when one drifts the agent gets contradictory signals.

!!! warning "One home per convention"
    Pick a single place for each rule. If it's in `AGENTS.md`, take it out of the rules folder. If it's in a skill, don't repeat it in `AGENTS.md`. Duplication looks like "being thorough" and behaves like a bug.

A simple heuristic: if the guidance applies to *every* task in this repo, put it in `AGENTS.md`. If it applies only to a file type or a subdirectory, use a scoped rule (if your agent supports it) or accept that a single short line in `AGENTS.md` is good enough.

## Why some agents deliberately don't have a rules system

Claude Code and Codex CLI made a conscious choice: no separate rules mechanism, everything goes into `CLAUDE.md`/`AGENTS.md`. The argument is that **fragmentation is the real enemy** — the more places the agent pulls instructions from, the harder it is to reason about why it did what it did.

The trade-off is real. You lose glob-scoped rules. You lose the ability to have a "TypeScript only" ruleset that silently activates when needed. In exchange, you get a single file to audit, a single file to review in PRs, and a single answer to "where did that behavior come from?".

If your project is small-to-medium and mostly one language, the single-file approach wins. If you have a polyglot monorepo where backend, frontend, and infra each want their own conventions, a dedicated rules system with glob scoping starts to pay for itself.

## Example: a small scoped Cursor rule

```markdown
---
description: TypeScript conventions for the web app
globs: ["apps/web/**/*.ts", "apps/web/**/*.tsx"]
alwaysApply: false
---

- Prefer `type` over `interface` unless declaration merging is needed.
- Never use `any`. If the type is truly unknown, use `unknown` and narrow.
- Components are function components; no classes.
- Props types live next to the component, named `<Component>Props`.
- Do not add new runtime dependencies without asking.
```

Notice the shape: frontmatter that tells the agent *when* to apply the rule, and a body that's short, declarative, and scannable. That's the format a rules system rewards.

## Key takeaway

!!! tip "Key takeaway"
    Rules, `AGENTS.md`, and skills are three answers to the same question: *how do I give this agent persistent guidance without retyping it?* Pick the mechanism your tool blesses, pick one home per convention, and resist the urge to copy the same rule into three files "just to be safe".
