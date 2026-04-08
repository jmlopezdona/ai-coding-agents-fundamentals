# 4. Slash commands and reusable prompts

When you find yourself typing the same prompt for the fifth time, it's time to crystallize it as a slash command (`/commit`, `/review-pr`, `/test`). Commands are how you turn one-off prompts into team conventions — a shared vocabulary that everyone, including the agent, agrees on.

## What you'll learn

- When a prompt deserves to become a command.
- The anatomy of a good command: scope, inputs, expected output.
- Where commands live and how they compound across a team.
- The failure modes that turn helpful commands into hidden landmines.

## The "fifth time" rule

The first time you write a prompt, you're exploring. The second and third time, you're refining. By the fifth time you've typed "look at the diff, write a conventional commit message, group related changes, and don't include co-author lines," you have stopped exploring — you have a recipe. Recipes belong in a file, not in your muscle memory.

!!! tip "Heuristic"
    If two teammates would write subtly different versions of the same prompt and get subtly different results, it's a command waiting to happen.

The same logic applies to Claude Code's `.claude/commands/`, Cursor's rules-as-commands, Codex prompts, or any Aider conventions file. The tool changes; the discipline doesn't.

## Anatomy of a good command

A good slash command behaves like a small pure function:

- **A clear name** — `/commit`, not `/do-the-git-thing`.
- **A single responsibility** — one verb, one outcome.
- **Explicit inputs** — what files, what arguments, what assumptions.
- **A predictable output** — the user should know what they'll get before they run it.

### Sample command file

Here is a minimal `/commit` command, written in the format most agents expect (Markdown with frontmatter and a body):

```markdown
---
description: Stage relevant changes and write a conventional commit message.
argument-hint: "[optional scope]"
---

You are preparing a commit for the current repository.

1. Run `git status` and `git diff` to see what changed.
2. Group related changes; ignore generated files and lockfiles unless they
   are the point of the change.
3. Write a Conventional Commits message:
   - type(scope): short summary (max 72 chars)
   - blank line
   - body explaining *why*, not *what*
4. Do NOT add Co-Authored-By lines.
5. Show me the message and the `git add` plan before committing.

Scope hint from the user: $ARGUMENTS
```

Notice what this command does *not* do: it doesn't push, it doesn't decide whether to amend, it doesn't open a PR. Those are separate commands.

## Where commands live

Most agents support two scopes:

- **Per-user** (e.g. `~/.claude/commands/`) — your personal toolbox.
- **Per-repo** (e.g. `.claude/commands/` checked into git) — the team's shared toolbox.

!!! note "Per-repo wins for teams"
    Personal commands are great for `/journal` or `/explain-this-error`. But anything that touches the codebase — commits, PRs, migrations, test runs — should live in the repo so reviewers can read it, change it, and trust it.

## Examples worth stealing

| Command | Responsibility |
|---|---|
| `/commit` | Stage and write a conventional commit. |
| `/review-pr <number>` | Pull a PR diff and review it against the project's standards. |
| `/test` | Run the right test command for the touched files and summarize failures. |
| `/explain <path>` | Produce a short architectural explanation of a file or module. |
| `/migrate <from> <to>` | Apply a known refactor (e.g. JUnit 4 → JUnit 5). |

Each one fits on a screen. Each one has a single job.

## Failure modes

!!! warning "The four ways commands rot"
    1. **Scope creep.** `/commit` starts pushing, then opens PRs, then runs tests. Split it.
    2. **Stale commands.** No one updated `/test` when the test runner changed. Treat commands like code: review, update, delete.
    3. **Hidden context.** A command that silently injects 4,000 tokens of "house style" makes results unreproducible. Be explicit about what it loads.
    4. **Magic names.** `/do-it` is not a command, it's a coin flip.

## Versioning and review

Commands are code. They live in git, they go through pull requests, and they deserve commit messages explaining why they changed. When `/review-pr` produces worse reviews next quarter, you want `git blame` to tell you who softened the rubric and why.

A useful rule: any change to a per-repo command should be reviewed by at least one person who *uses* that command, not just the person who wrote it. Commands shape behavior across the whole team — they deserve the same care as a shared library.

## Key takeaway

!!! success "Key takeaway"
    A slash command is a function call for prompts. Apply the same hygiene you'd apply to any function: single responsibility, clear interface, code review, and a name you wouldn't be embarrassed to read out loud in a standup.
