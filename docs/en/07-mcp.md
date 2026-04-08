# 7. MCP and external tools

MCP (Model Context Protocol) is a standard way to plug external tools and data sources into an agent: a database, a ticketing system, a docs site, a custom internal API. This chapter explains it without the jargon and helps you decide when MCP is worth it vs a plain script.

## What you'll learn

- What MCP is, in one page.
- When to add an MCP server vs a local script or a CLI tool.
- Basic supply-chain risks of installing third-party MCP servers.

## Outline

1. The problem MCP solves: every agent needs the same integrations, every vendor was inventing its own.
2. The model: server exposes tools/resources/prompts, agent discovers and calls them.
3. When MCP is worth it: shared across team, multiple agents, needs auth, needs to be discoverable.
4. When a script or CLI tool is enough: one-off, single user, simple input/output.
5. Supply chain: who wrote this MCP server? what does it have access to? prompt injection through tool results.
6. A short list of MCP servers that pay off early (filesystem, GitHub, your docs).

## Key takeaway

MCP is plumbing, not magic. Use it when integration needs to be shared and discoverable; reach for a script when it doesn't.
