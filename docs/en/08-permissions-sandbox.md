# 8. Permissions, sandbox, and execution modes

Every agent product has knobs for "how much can it do without asking?" Get this wrong in either direction and you'll suffer: too restrictive and the agent is useless, too permissive and you'll regret it the first time it runs `rm -rf` or pushes to `main`.

## What you'll learn

- The execution modes most agents offer (plan, ask, auto-accept).
- How to choose autonomy per tool, not per session.
- A safe default for a team starting out.

## The autonomy spectrum

Every harness — Claude Code, Cursor, Codex, Aider, Continue — exposes roughly the same dial, even if the labels differ:

- **Plan mode** — the agent may read and think, but cannot edit files or run commands. It produces a plan you review before anything changes. Ideal for exploring a new codebase or scoping a refactor.
- **Ask (default) mode** — every tool call that mutates state (write, bash, network) prompts you for approval. Slow but safe.
- **Accept-edits mode** — file edits auto-apply, but commands and network calls still ask. A good middle ground once you trust the agent with a task.
- **Bypass / full-auto mode** — the agent runs without confirmation. Useful in sandboxes and CI. Dangerous on your laptop.

!!! note "Different names, same idea"
    Claude Code calls these plan/default/accept-edits/bypass. Cursor has agent modes. Codex CLI has `suggest`/`auto-edit`/`full-auto`. The semantics map cleanly across products.

## Why full-auto for everything is a beginner trap

The first time you flip an agent to full-auto on real code, it feels magical. The tenth time, it has force-pushed a branch, deleted your virtualenv, or opened twelve PRs in the wrong repo. The cost of one extra confirmation click is a second. The cost of one unwanted destructive command can be hours of recovery and lost trust.

Full-auto is a tool for environments where mistakes are cheap: ephemeral containers, throwaway branches, scratch directories. It is not a default for your main working tree.

## Per-tool permissions

The better mental model is not "how much do I trust this session" but "how much do I trust this capability." Permissions should be set per tool:

- **Always allow:** `Read`, `Grep`, `Glob`, `LS`. Read-only operations can't corrupt state. Letting the agent grep freely makes it dramatically more useful.
- **Auto-allow with scope:** `Edit`, `Write` within the project directory. Block writes outside the repo.
- **Allowlist:** `Bash`. Permit specific commands (`pnpm test`, `pnpm lint`, `git status`, `git diff`) without confirmation; prompt for anything else.
- **Always ask:** destructive git (`push`, `reset --hard`, `checkout .`), package installs, network calls to unknown hosts, anything touching `~/.ssh` or `.env`.

A per-tool allowlist gives you most of the speed of full-auto with almost none of the blast radius.

## Sandboxing and trade-offs

Permissions decide what the agent is allowed to try. Sandboxes decide what happens when it tries something anyway. The common options:

- **Git worktrees.** Cheapest isolation. The agent works in a separate working copy of the same repo, so an accidental delete or bad commit doesn't touch your main checkout. Fast, local, no container overhead.
- **Containers (Docker, devcontainers).** The agent's filesystem view and network are scoped to a container. Strong isolation at the cost of slower startup and a more awkward edit/debug loop.
- **Ephemeral VMs or cloud sandboxes.** Services like GitHub Codespaces, Daytona, E2B, or vendor-hosted agent sandboxes. Best isolation, highest latency, extra cost. The right fit for autonomous long-running agents.
- **No sandbox.** Running directly on your laptop. Fastest feedback, zero isolation — lean heavily on permissions and hooks.

!!! warning "Sandboxes aren't free"
    Every layer of isolation adds friction: slower hot reloads, harder debugger attach, confusing path mismatches. Pick the lightest sandbox that actually addresses your risk. For most day-to-day work in a trusted repo, a worktree plus good hooks beats a heavyweight container.

## Safe defaults for your first month

If you're setting up an agent for a team that has never used one:

1. Start in **ask mode**, per-tool.
2. Auto-allow `Read`, `Grep`, `Glob`, `LS`.
3. Auto-allow `Edit` and `Write` inside the project directory only.
4. Allowlist safe `Bash` commands: your test runner, linter, formatter, `git status`, `git diff`, `git log`.
5. Require confirmation for any `git push`, any `rm`, any package install, any network fetch.
6. Run inside a git worktree so "undo" is `git worktree remove`.
7. Enable the day-one hooks from chapter 6 so formatting and secret scanning happen automatically.

After two weeks of real use, you'll know which prompts you always approve — promote those to the allowlist. You'll also know which tools the agent keeps misusing — tighten those.

### When to relax — deliberately

Relax permissions because you learned something, not because you're frustrated. Good reasons:

- "I've accepted this command 50 times in a row without surprises" → allowlist it.
- "This task runs in a sandbox that I can throw away" → bypass mode is fine there.
- "CI runs the agent on a disposable branch" → full-auto is correct.

Bad reasons:

- "The confirmations are annoying and I'm in a hurry."
- "It's a small change, what could go wrong."

!!! tip "Promote, don't bypass"
    When a confirmation gets annoying, add the command to your allowlist. Don't switch the whole session to bypass mode — that loses the safety for every other tool too.

!!! success "Key takeaway"
    Choose autonomy per tool, not per session. The cost of one prompt-confirmation is tiny; the cost of one unwanted `git push --force` is enormous.
