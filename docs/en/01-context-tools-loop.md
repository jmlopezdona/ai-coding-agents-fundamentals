# 1. Context, tools, loop — the mental model

Every coding agent, regardless of vendor, is made of the same three pieces: **what it sees** (context), **what it can do** (tools), and **how it decides** (loop). If you internalize these three, you can reason about any agent product without re-learning the mental model.

## What you'll learn

- The three pieces every agent has, and how they interact.
- How to ask "what does the agent see right now?" and "what can it actually do?" before debugging weird behavior.
- Why most agent failures are context failures or tool failures, not model failures.

## Context: what the model sees this turn

Context is everything that ends up in the model's input on a given turn. It is not "the project" — the model has no magical view of your repo. It only knows what has been explicitly placed into its context window.

A typical turn's context includes:

- **System prompt** — set by the harness (Claude Code, Cursor, Codex, Aider...). Defines the agent's role, available tools, safety rules.
- **Project instructions** — your `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, etc. (Chapter 2.)
- **User message** — what you typed.
- **Conversation history** — previous user messages, model responses, tool calls, tool results.
- **Files read so far** — every `read_file` result is now part of context.
- **Search results, command outputs** — same.

```
[system prompt]
[AGENTS.md]
[user] "fix the failing test in billing_test.py"
[assistant] read_file(billing_test.py)
[tool_result] <contents of billing_test.py>
[assistant] read_file(billing.py)
[tool_result] <contents of billing.py>
[assistant] run_shell("pytest billing_test.py")
[tool_result] FAILED ... AssertionError: 10.05 != 10.04
[assistant] ...thinking about the fix
```

!!! note "Context windows are finite"
    Even with 200k or 1M token windows, context is a budget. Reading a 50k-line file consumes that budget. Good agents (and good users) read narrowly: search first, open the relevant slice, not the whole file.

## Tools: the verbs the agent has

Tools are the agent's verbs. Without them, the model can only talk. With them, it can act. A typical coding agent harness exposes some subset of:

- **Filesystem**: `read_file`, `write_file`, `edit_file`, `list_dir`, `glob`.
- **Search**: `grep`, semantic search, symbol lookup.
- **Execution**: `run_shell` (often sandboxed), `run_tests`.
- **Network**: `fetch_url`, `web_search` (sometimes).
- **VCS**: `git_diff`, `git_commit` (sometimes).
- **MCP servers**: arbitrary user-provided tools (databases, Jira, internal APIs).

Different products expose different tools. Cursor leans on its own indexed search. Aider uses git aggressively. Claude Code exposes a small, sharp set of tools with explicit permissions. Codex CLI runs everything in a sandbox by default. The *names* differ; the *categories* are nearly universal.

!!! warning "Tools you don't have, the agent can't do"
    If your agent has no `run_shell`, it cannot run your tests, no matter how clearly you ask. If it has no web access, it cannot check current docs. Knowing the toolset is half of knowing the agent.

## The loop: how decisions get made

The loop is deceptively simple:

```
while not done:
    response = model(context)
    if response.has_tool_calls:
        results = harness.execute(response.tool_calls)
        context.append(response, results)
    else:
        return response   # final answer to user
```

That's it. The model doesn't have a hidden planner. It doesn't have a debugger watching it. Each turn, it sees the accumulated context and decides: call another tool, or stop and answer. The "intelligence" of an agent run is the sum of many small local decisions, each one made on the context that exists at that moment.

### A single turn vs a full task

A **turn** is one round of model output. A **task** is the full sequence of turns from your prompt to the agent's final answer. A trivial task ("rename this variable") might be 2 turns. A real task ("add a feature with tests") might be 30. Each turn is independent in the sense that the model re-reads the entire context from scratch — it has no persistent memory between turns beyond what's in that context.

## Diagnosis: was it context, tools, or loop?

When an agent does something stupid, resist the urge to immediately rewrite your prompt. Walk the three layers in order.

!!! tip "The three diagnostic questions"
    1. **Context** — Did it see the right things? Did it read the file it was supposed to edit? Did it have your `AGENTS.md`? Did older turns push the relevant info out of the window?
    2. **Tools** — Did it have the verb it needed? If it "guessed" at code instead of reading the file, maybe it had no working `read_file`. If it didn't run the tests, maybe `run_shell` is disabled.
    3. **Loop** — Did it terminate too early (gave up before verifying)? Did it loop forever on a flaky tool? Did it hit a tool error and give up instead of retrying?

A few concrete examples:

- *"It hallucinated a function that doesn't exist."* Almost always context: it never read the file where the function lives.
- *"It says the test passes but it doesn't."* Almost always loop or tools: it never actually ran the test, or it ran a different one.
- *"It edited the wrong file."* Context: it didn't search first; it guessed the path.

Prompt engineering is the *last* lever, not the first. Fixing context (point it at the right files, trim noise, add an `AGENTS.md` line) and fixing tools (give it shell access, give it a real test runner) solves the vast majority of agent misbehavior.

## Closing the loop: verification before handing back

The loop in the pseudocode above terminates when the model decides to stop calling tools. But "the model stopped" is not the same as "the task is done". A loop is only really closed when the agent has *verified its own work* against a check that you both understand — tests pass, lint is clean, the typechecker is silent, the dev server returns the expected response, the screenshot matches.

This has a consequence for how you write prompts. Before you delegate anything, ask yourself: *"how will I know this is done?"* If you can't answer, the agent can't either, and you'll spend the review acting as a human test runner. If you can answer, put the answer in the prompt — and make sure the agent has the tools (a real shell, a test command, a linter, a typechecker) to run that check itself and iterate until it passes. The metric that matters is not "did the agent produce a diff" but "what percentage of agent tasks pass my first review without a round-trip". Self-verification is what moves that number.

!!! tip "Define your acceptance check before you write the prompt"
    If you can't say how you'd verify success, the agent can't either. Decide the check first, expose the tool that runs it, and require the agent to run it before handing control back.

## Key takeaway

!!! tip "Key takeaway"
    When something goes wrong, ask the three questions in order: did it see the right things? did it have the right tools? did the loop terminate cleanly? This beats prompt-tweaking nine times out of ten.
