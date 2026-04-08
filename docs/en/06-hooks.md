# 6. Hooks and automation

Hooks are commands the harness runs automatically on events (before a tool call, after an edit, before a commit). They're how you enforce things the agent shouldn't be trusted to remember — formatting, linting, secret scanning, test runs.

## What you'll learn

- The difference between "asking the agent to do X" and "making the harness enforce X."
- Common hooks worth setting up on day one.
- Why hooks beat instructions for anything safety-critical.

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
2. **Linter on edit.** ESLint, Ruff, golangci-lint. Catches the agent's favorite mistakes (unused imports, shadowed variables).
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
          },
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_FILE_PATHS\""
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

The `PostToolUse` hook runs Prettier then ESLint on any file the agent just wrote. The `PreToolUse` hook inspects bash commands and exits non-zero to block things like `rm -rf /` or `git push --force` to `main`.

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
    If it must always happen, it must be a hook. Instructions are hopes; hooks are guarantees.
