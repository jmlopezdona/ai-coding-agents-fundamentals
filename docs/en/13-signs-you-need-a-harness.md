# 13. Signs you need a harness

This guide is enough to get your team productive with coding agents. It is *not* enough to keep you productive once you scale beyond a handful of developers or a handful of repositories. At some point you will feel a kind of friction that no amount of prompt-tweaking fixes. That is the moment to graduate from "using an agent" to specifying intent properly (**SDD**) and, after that, building the system that sustains it (**harness engineering**).

## What you'll learn

- What a harness actually is.
- Concrete symptoms that mean "you've outgrown this guide."
- Why the fix is structural, not prompt-level.
- Where to go next.

## What is a "harness"?

The model is only one part of a coding agent. Everything *around* the model — the runtime that shapes what it sees, what it can do, and what happens to its output — is the harness. Concretely:

- **Hooks.** Deterministic code that runs before or after tool calls (`PreToolUse`, `PostToolUse`, `Stop`). This is where you enforce rules, run linters, block dangerous commands, and inject context.
- **Sandbox.** The environment the agent runs in — devcontainer, Docker, a VM, a restricted shell. Defines what it can read, write, and reach on the network.
- **Subagents.** Specialized sub-processes with their own prompts and tool access: a reviewer, a test-runner, a researcher. They let you decompose work and keep contexts small.
- **Memory.** The layered system from chapter 11 — project memory, user memory, skills, commands.
- **MCP and tools.** The catalog of tools the agent can call: file system, shell, search, custom MCP servers for your internal systems.
- **Guides and sensors.** Documents that teach the agent *how* to do something, and checks that detect when it has gone off the rails.

A harness is not a product — it's an assembly. Claude Code, Cursor, Codex, and Aider each ship with a default harness. For a while, the default is enough. Then it isn't.

It's worth making this explicit: what you learn in *Fundamentals* is, in fact, how to use the default harness those tools ship with — Claude Code's, Cursor's, Copilot's, Codex's. For many teams and many projects, that default is enough, and you should squeeze every drop out of it. When we say "you need a harness," we don't mean you weren't using one before — we mean the generic one falls short of the specifics of *your* system: your rules, your domain, your integrations, your criticality level. You need a *tailored* one — a harness built for the concrete system you're creating, maintaining, modernizing, or refactoring. And yes, over time the default harnesses will absorb more and more capabilities and the frontier of necessary tailoring will keep narrowing — but as long as that frontier exists, someone on your team has to cross it.

That crossing is not direct. Before investing in building your own harness, you almost always want to go through **Spec-Driven Development (SDD)** first: learning to specify intent clearly and to keep that spec alive as the code evolves. The reason is practical — a more sophisticated harness gives the agent more autonomy, and autonomy without a clear spec amplifies noise instead of productivity. SDD is the bridge: it forces you to write down *what* you want before you automate the *how*. Only then does the next jump make sense — **harness engineering**: bringing engineering discipline to building the harness that lets you implement systems with a high degree of autonomy, at scale, and across the entire SDLC. Fundamentals → SDD → Harness Engineering is the path.

## Symptoms that you've outgrown the default

Here is the checklist. If more than two or three of these feel familiar, you're past the point where better prompts will help.

### The agent repeats the same mistakes across developers

Alice teaches her session not to touch the migrations folder. Bob's session does it anyway the next day. There is no shared correction loop — every developer is privately retraining the same agent.

**What's missing:** a shared rule, enforced in the harness (hook or permission), not in one person's prompt.

### Every developer has their own setup

Walk around the team. Alice uses Claude Code with her own `~/.claude` tree. Bob uses Cursor with custom rules nobody else has seen. Carol uses Codex with a different model entirely. None of their configs are in version control.

**What's missing:** a repo-level, versioned harness. Shared commands, shared skills, shared hooks.

### Agent PRs need more review than human PRs

You are spending more time reviewing the agent's output than you would have spent writing it yourself. Verification has been pushed onto the human instead of the harness.

**What's missing:** automated sensors — linters, tests, type checks, security scanners — running *before* the PR reaches a human reviewer, and a feedback loop that makes the agent fix its own output.

### The same class of bug keeps coming back

You review three PRs in a week and all three have the same mistake: a missing null check, a wrong transaction boundary, a hardcoded secret. You correct it each time. It comes back.

**What's missing:** a sensor that catches this class of bug automatically, plus a guide the agent loads whenever it touches the relevant code.

### Onboarding a new dev to "the agent way" takes a week

There is no document you can point a new hire to. The knowledge lives in heads and private chat logs. Every new person rediscovers the same lessons.

**What's missing:** a system of record. The harness *is* the onboarding — if it lives in the repo, new hires inherit it on day one.

### Your prompts are getting longer and longer

You notice you've started every session with a 20-line preamble: "Remember to run the tests, remember the migration rules, remember not to touch X, remember Y, remember Z..." You are doing by hand what a harness should do automatically.

**What's missing:** guides loaded on demand, hooks that enforce rules, and context injected by the harness instead of by you.

### You're afraid to let the agent run unattended

Every session is babysat. You never kick off a task and come back in 30 minutes. The risk of the agent doing something destructive feels too high.

**What's missing:** a real sandbox, a real permission model, and sensors that can stop a run when something looks wrong.

### You're using the agent across multiple SDLC phases

When the same agent helps with design, code, QA, and ops, you need shared memory, skills, and MCP servers that travel with it — not three disconnected setups.

**What's missing:** a single harness that carries context, skills, and tool access across phases, so the agent doesn't lose everything it knows the moment the task changes shape.

## Why prompt-engineering won't fix any of this

All of these symptoms share a root cause: **the problem is not what the model is told, it's the absence of a system around the model.** A longer prompt cannot replace a hook. A better memory file cannot replace a sandbox. A clever instruction cannot replace an automated sensor. Once you're trying to use words to fix structural gaps, you are fighting the wrong battle.

## What's next

When you recognize these symptoms, jump to the companion guides. This guide is the first volume of a trilogy; the other two pick up from different angles:

!!! info "Spec-Driven Development"
    Next up: **[Spec-Driven Development](https://jmlopezdona.github.io/ai-coding-agents-sdd/)**

    The bottleneck is no longer writing code — it's specifying intent clearly and keeping that spec alive as the code evolves. This guide covers the loops and patterns that make the agent consistently build what you actually meant — and when that discipline helps versus when it turns into bureaucracy.

!!! info "Harness Engineering"
    And after that: **[Harness Engineering — building software at scale with agents](https://jmlopezdona.github.io/ai-coding-agents-harness/)**

    It picks up exactly where this one stops: how to build the guides, sensors, loops, sandboxes, subagents, and structured context that turn an LLM into an agent a team can truly rely on.

!!! success "Key takeaway"
    If you're tweaking prompts to fix structural problems, you've outgrown this guide. The next step isn't a better prompt: it's learning to specify intent properly (SDD) and, when that no longer scales, building the harness that sustains it.
