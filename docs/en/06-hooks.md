# 6. Hooks and automation

Hooks are commands the harness runs automatically on events (before a tool call, after an edit, before a commit). They're how you enforce things the agent shouldn't be trusted to remember — formatting, linting, secret scanning, test runs.

## What you'll learn

- The difference between "asking the agent to do X" and "making the harness enforce X."
- Common hooks worth setting up on day one.
- Why hooks beat instructions for anything safety-critical.

## Outline

1. The unreliability of "please always run the linter" instructions.
2. Hook events: pre-tool, post-tool, pre-commit, session-start, session-end.
3. Day-one hooks: formatter, linter, secret scanner, test runner on changed files.
4. Hooks that block vs hooks that warn.
5. Debugging hook failures without bypassing them (`--no-verify` is a smell).
6. The bridge to the harness guide: hooks are where harness engineering really begins.

## Key takeaway

If it must always happen, it must be a hook. Instructions are hopes; hooks are guarantees.
