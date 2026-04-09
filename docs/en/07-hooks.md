# 7. Hooks and automation

Hooks are commands the harness runs automatically on events (before a tool call, after an edit, before a commit). They're how you enforce things the agent shouldn't be trusted to remember — formatting, linting, secret scanning, test runs.

## What you'll learn

- The difference between "asking the agent to do X" and "making the harness enforce X."
- Common hooks worth setting up on day one.
- Why hooks beat instructions for anything safety-critical.
- When to run things per-edit versus at end-of-turn.
- How hooks act as sensors that feed failures back into the agent's context.

## Instructions are hopes, hooks are guarantees

Every team that adopts an AI coding agent goes through the same cycle. Someone writes in `AGENTS.md` or `CLAUDE.md`: "Always run `pnpm lint` after editing TypeScript." For a while it works. Then the model changes, the context gets long, the task gets urgent, and the instruction quietly stops being followed. The agent ships code that fails CI.

The lesson is simple: a system prompt is a suggestion the model is free to ignore under pressure. A hook is a command the harness runs whether the model likes it or not. If a step must always happen, it cannot live in prose — it has to live in configuration.

!!! note "Applies to every harness"
    Claude Code, Cursor, Codex CLI, Aider and others all expose some form of event hook or pre/post command. The names differ but the principle is identical: the harness enforces, the model requests.

## Hook events you'll actually use

Most harnesses expose a similar event surface. The useful ones are:

- **PreToolUse** — fires before a tool call. Can block it. Use for policy ("don't let the agent edit `/etc`") and redaction.
- **PostToolUse** — fires after a tool call succeeds. Use for formatting, linting, and running fast tests on changed files.
- **PreCommit / PrePush** — classic git hooks, still valuable. Secret scanning lives here.
- **SessionStart / SessionEnd** — good place to log, snapshot state, or print the current branch and dirty files so the agent starts grounded.
- **UserPromptSubmit** — fires when the user sends a message, useful for injecting context or blocking obviously dangerous asks.

## Day-one hooks

Before you tune prompts, set up these four. They pay for themselves within a day:

1. **Formatter on edit.** Prettier, Black, gofmt, rustfmt. Removes a whole class of pointless diffs.
2. **Linter / typechecker at end-of-turn.** ESLint, Ruff, golangci-lint, tsc, mypy. Catches the agent's favorite mistakes (unused imports, shadowed variables, broken types) — but run them once when the turn closes, not on every edit (see "Per-edit vs end-of-turn" below).
3. **Secret scanner pre-commit.** gitleaks or trufflehog. The agent will eventually paste an API key somewhere.
4. **Affected tests.** Run the tests that touch changed files. Fast feedback, not full CI.

### A sample `settings.json` snippet

Here's a Claude Code-style hooks configuration. Cursor and Codex use different schemas but the shape is similar.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATHS\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint . && npx tsc --noEmit"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/deny-dangerous-commands.sh"
          }
        ]
      }
    ]
  }
}
```

The `PostToolUse` hook runs Prettier on any file the agent just wrote — cheap, idempotent, no failures. The `Stop` hook runs ESLint and `tsc` once when the agent finishes its turn, producing a single coherent verification pass instead of noise after every edit. The `PreToolUse` hook inspects bash commands and exits non-zero to block things like `rm -rf /` or `git push --force` to `main`.

## Per-edit vs end-of-turn: when to run what

Not every check belongs on every edit. The right rule of thumb:

- **Per-edit (`PostToolUse`)** is for **formatters**: Prettier, Black, gofmt, rustfmt, ruff format. They're cheap, idempotent, can't really "fail", and they keep the agent from ever seeing badly-formatted code. Running them on every write is the correct default.
- **End-of-turn (`Stop` / `SubagentStop`)** is for **linters, typecheckers, and tests**: ESLint, mypy, tsc, pytest. They're expensive, noisy, and frequently fail mid-refactor — a symbol may be temporarily missing, an import temporarily broken. Running them once at the end of the turn over everything that changed is faster and produces a single, coherent feedback signal.

The useful rule: **format per-edit, validate at end-of-turn.**

!!! warning "Heavy validation per-edit hurts more than it helps"
    Running `tsc` after every `Edit` wastes tokens, slows the loop, and floods the agent with transient errors from work-in-progress code. The agent then "fixes" things that weren't broken — they were just half-done. Wait until the turn closes.

## Hooks as sensors: how the agent learns about failures

A hook that runs is only half the story. The other half is: **does the agent find out it ran, and what it found?**

The answer lives in the hook's exit code:

- **Exit 0** — silent. The agent continues as if nothing happened.
- **Non-zero exit (or stderr)** — the harness typically injects the hook's stderr back into the agent's context as a system message. The agent sees it as if it were tool feedback, opens the file, reads the error, fixes it, and retries. The verification loop closes without human intervention.
- **`PreToolUse` hooks can be blocking**: a non-zero exit aborts the action *before* it happens. This is how guardrails like "don't `rm -rf` outside the workspace" actually work.

This is the bridge to chapter 1's "verification before handing back". A well-designed hook isn't just an automator — it's a **sensor** that turns side-effects into feedback the agent can act on. The end-of-turn `tsc` failure becomes the next turn's first task. The agent self-corrects.

!!! warning "The anti-pattern: hooks that only log"
    A hook that pipes output to `/tmp/agent-hooks.log` and exits 0 is decoration. Nothing flows back into the agent's context, so failures are invisible until a human reads the log. If you want the agent to react, the failure must be visible to it.

!!! tip "A hook isn't valuable because it ran"
    It's valuable because the agent noticed it ran and acted on the result. Wire the exit code with intention.

## Blocking vs warning

Not every hook should fail the tool call. Roughly:

- **Block** on policy violations, secrets, dangerous commands, broken syntax.
- **Warn** (log but don't fail) on style nits, slow tests, or advisory lint rules.

A hook that blocks too aggressively trains the agent — and the human — to look for bypasses. A hook that only warns gets ignored. Pick the right severity per rule.

!!! warning "`--no-verify` is a smell"
    If you find yourself or the agent reaching for `git commit --no-verify`, the correct response is almost never "bypass the hook." It's either "fix the underlying problem" or "the hook is wrong, fix the hook." Bypassing is how guardrails rot.

## Debugging hook failures

When a hook fails, resist the urge to skip it. Instead:

1. Run the hook command manually in your shell with the same inputs.
2. Check that environment variables the harness injects (like `$CLAUDE_FILE_PATHS`) are what you expect.
3. Make the command idempotent — hooks that fail on re-run are painful.
4. Keep hooks fast. A 30-second hook on every edit will destroy the flow.

!!! tip "Log, don't guess"
    Pipe hook output to a logfile (`>> /tmp/agent-hooks.log 2>&1`) during setup. You'll debug in minutes instead of hours.

## The bridge to harness engineering

Hooks are where "using an agent" becomes "engineering a harness." Once you have a formatter, a linter, a secret scanner, and a test runner wired in, you stop worrying about whether the agent remembered the rules — because the rules aren't the agent's job anymore. That shift of responsibility, from prompt to plumbing, is the single biggest maturity jump a team makes.

!!! success "Key takeaway"
    Hooks have two jobs: **automate** (do the thing the agent shouldn't be trusted to remember) and **sense** (feed failures back so the agent self-corrects). If it must always happen, it must be a hook — and if you want the agent to react, the hook must speak to it through its exit code.
