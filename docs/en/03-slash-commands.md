# 3. Slash commands and reusable prompts

When you find yourself typing the same prompt for the fifth time, it's time to crystallize it as a slash command (`/commit`, `/review-pr`, `/test`). Commands are how you turn one-off prompts into team conventions.

## What you'll learn

- When a prompt deserves to become a command.
- Anatomy of a good command: scope, inputs, expected output.
- How commands compound across the team.

## Outline

1. The "fifth time" rule: prompts you repeat become commands.
2. Anatomy: a clear name, a single responsibility, explicit inputs, predictable output.
3. Examples: `/commit`, `/review-pr`, `/test`, `/explain`, `/migrate`.
4. Where commands live (per-user vs per-repo) and why per-repo wins for teams.
5. Failure modes: commands that try to do too much, commands no one updates, commands that hide important context.
6. Versioning and review — yes, commands are code too.

## Key takeaway

A slash command is a function call for prompts. Apply the same hygiene: single responsibility, clear interface, code review.
