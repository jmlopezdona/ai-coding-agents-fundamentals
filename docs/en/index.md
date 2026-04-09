# AI Coding Agents — Fundamentals

A practical, vendor-neutral guide to working seriously with AI coding agents — before you need a full harness around them.

## Who this guide is for

Technical teams that are *just starting* to operate AI coding agents (Claude Code, Codex, Cursor, Copilot…) and want a solid mental model before they hit scaling pain. No prior experience with agents required — only with software engineering.

!!! warning "This space moves fast"
    AI coding agents evolve in weeks, not years. A feature this guide describes as "missing" in tool X may already exist by the time you read it; a vendor-specific name may have been renamed; a "Yes/No" comparison row may have flipped. Treat the comparisons as a snapshot of intent, not as an authoritative spec — and check each tool's current docs before making decisions. The **mental model** (context, tools, loop, verification, hooks, skills, subagents) is what stays stable; the implementations don't.

## How to read it

The 14 chapters build progressively. If you're brand new, read in order. If you already have some experience, jump to the chapter that matches your current question.

## Chapters

- [0. What an AI coding agent really is](00-what-is-an-agent.md)
- [1. Context, tools, loop — the mental model](01-context-tools-loop.md)
- [2. Your first AGENTS.md](02-first-agents-md.md)
- [3. Rules and instructions: same problem, different formats](03-rules-instructions.md)
- [4. Slash commands and reusable prompts](04-slash-commands.md)
- [5. Skills: knowledge on demand](05-skills.md)
- [6. Subagents and delegation](06-subagents.md)
- [7. Deterministic guarantees: hooks, specialized agents, and external workflows](07-hooks.md)
- [8. MCP and external tools](08-mcp.md)
- [9. Permissions, sandbox, execution modes](09-permissions-sandbox.md)
- [10. Privacy and sensitive data](10-privacy-sensitive-data.md)
- [11. Memory and persistence across sessions](11-memory-persistence.md)
- [12. Anti-patterns of the first months](12-anti-patterns.md)
- [13. Signs you need a harness](13-signs-you-need-a-harness.md)

## What's next

This guide is the first volume of a trilogy. When it stops being enough — when your team starts feeling the pain of running agents at scale — the next steps are:

👉 **[Spec-Driven Development](https://jmlopezdona.github.io/ai-coding-agents-sdd/)** — how to specify intent clearly and keep that spec alive as the code evolves.

👉 **[Harness Engineering — building software at scale with agents](https://jmlopezdona.github.io/ai-coding-agents-harness/)** — how to build the guides, sensors, loops, sandboxes, subagents, and structured context that turn an LLM into an agent a team can rely on.
