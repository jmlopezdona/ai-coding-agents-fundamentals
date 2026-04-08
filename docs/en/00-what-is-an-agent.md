# 0. What an AI coding agent really is

Most developers met LLMs through autocomplete (Copilot) or chat (ChatGPT). An *agent* is neither. The qualitative jump is not the model — it's the **loop**: a model that can read files, run tools, observe results, and decide what to do next, repeatedly, until a task is done.

## What you'll learn

- The difference between autocomplete, chat, and agent.
- Why the loop — not the model — is what makes the difference.
- A first vocabulary: tools, context, turn, tool result.

## Three machines, not one

It's tempting to think of Copilot, ChatGPT, and Claude Code as three flavors of "the same AI thing." They aren't. They are three architecturally distinct systems that happen to share an LLM at their core. Confusing them is the root cause of most bad takes about AI coding tools — both the hype and the dismissal.

### Autocomplete: predict the next token

Tools like GitHub Copilot's inline completion, Tabnine, or early Codex behave like a very smart text predictor. You type, the model proposes the next few tokens, you accept or reject. There is no memory between completions, no tools, no decisions. The model never reads a file you didn't open, never runs a test, never asks itself "did that work?"

```python
def parse_iso_date(s: str) -> datetime:
    # cursor here — autocomplete suggests the body
```

The unit of work is **a suggestion**. The reviewer is **you, every keystroke**.

### Chat: multi-turn conversation

ChatGPT, Claude.ai, Gemini in a browser tab — these are conversational. They keep the dialogue history, can reason across turns, and can produce long structured outputs. But by default they still cannot touch your filesystem, run your tests, or check whether the snippet they just wrote actually compiles. You copy-paste in, you copy-paste out.

The unit of work is **an answer**. The reviewer is **you, after each reply**.

### Agent: model + tools + loop

Claude Code, Cursor's agent mode, OpenAI Codex CLI, Aider, Cline, Continue — these are different. The model is wired to a set of **tools** (read file, write file, run shell command, search code, fetch URL...) and runs inside a **loop**: it emits a tool call, the harness executes it, the result comes back as a new message, and the model decides what to do next. It keeps going until it believes the task is done — or it asks you a question.

```
user:    "the test in billing_test.py is failing, fix it"
agent:   read_file(billing_test.py)
agent:   read_file(billing.py)
agent:   run_shell("pytest billing_test.py")     -> red
agent:   edit_file(billing.py, ...)
agent:   run_shell("pytest billing_test.py")     -> green
agent:   "fixed: rounding was using floor instead of round-half-even"
```

The unit of work is **a task**. The reviewer is **you, on the diff and the result**.

## The same task, three ways

Imagine: *"the test `test_invoice_total` is failing, fix it."*

- **Autocomplete** can't help. It doesn't know the test exists.
- **Chat** can help if you paste the test, the failing function, and the stack trace — and it gives you back code you have to paste, save, and re-run yourself. You are the loop.
- **Agent** opens the test, reads the function, runs the test to see the actual failure, edits the code, re-runs the test, and reports back. The loop is automated.

That last sentence is the whole point of this guide.

## A first vocabulary

!!! note "Words you'll see in every chapter"
    - **Context** — everything the model can see right now: your message, system prompt, files it has read, previous tool results.
    - **Tool** — a function the agent is allowed to call (e.g. `read_file`, `run_shell`).
    - **Tool call** — the model's request to invoke a tool, with arguments.
    - **Tool result** — what comes back, fed into the next turn.
    - **Turn** — one round of "model thinks → model emits something → harness reacts."
    - **Harness** — the program around the model that actually executes tools, manages files, enforces permissions. Claude Code, Cursor, Aider, etc. *are* harnesses.

## Why this changes how you review code

When you use autocomplete, you review at the keystroke level. When you use chat, you review at the snippet level. When you use an agent, you review at the **task level** — the diff plus the evidence the agent gathered (which tests it ran, which files it touched, what it observed).

!!! warning "Don't review an agent like a junior developer's PR"
    You can't catch every line. Instead: check the diff is small and focused, check the tests it claims to have run actually exist and pass, and treat anything outside the requested scope as a red flag.

This is also why "the AI made a mess" stories almost always come from people using an agent as if it were autocomplete: accepting suggestions one at a time, never looking at the overall diff, never asking what tests were run.

## Beyond code: agents across the SDLC

"Coding agent" is the most visible use case — it's where the tooling is sharpest and the feedback loop is fastest — but the loop itself is general. Anywhere a task can be decomposed into steps with verifiable outcomes, the same mental model applies: **context + tools + loop + verification**. Swap "compile and run tests" for "validate against an OpenAPI schema" or "diff a Terraform plan" and the machine still works.

A non-exhaustive tour of the SDLC:

- **Discovery / product** — synthesizing user interviews into themes, drafting user stories from raw notes.
- **Design / UX / UI** — pulling components from a design system via an MCP server to Figma, running accessibility audits, generating mockups from a brief.
- **Estimation and planning** — breaking down epics into stories, estimating by analogy from historical tickets in Jira.
- **Functional and technical design** — drafting ADRs, sketching C4 diagrams, writing OpenAPI specs, proposing data models.
- **Coding and unit testing** — the default case this guide focuses on.
- **QA** — functional and exploratory testing, security scanning (SAST/DAST), performance testing, generating realistic test data.
- **Deployment** — authoring CI/CD pipelines, writing IaC, reviewing Terraform PRs against policy.
- **Operations** — incident triage, log and trace analysis, executable runbooks, on-call assistance.

Each phase has its own tools, its own verification signal, and its own definition of "done" — but the loop is the same loop.

The rest of this guide uses coding examples because they're concrete and easy to verify, but everything you'll learn about — AGENTS.md, skills, hooks, MCP, verification loops — generalizes to every phase above.

## Key takeaway

!!! tip "Key takeaway"
    Stop comparing agents to autocomplete. They're different machines. The unit of work is no longer "a suggestion you accept" — it's "a task you delegate and verify."
