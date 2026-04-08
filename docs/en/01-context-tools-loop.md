# 1. Context, tools, loop — the mental model

Every coding agent, regardless of vendor, is made of the same three pieces: **what it sees** (context), **what it can do** (tools), and **how it decides** (loop). If you internalize these three, you can reason about any agent product without re-learning the mental model.

## What you'll learn

- The three pieces every agent has, and how they interact.
- How to ask "what does the agent see right now?" and "what can it actually do?" before debugging weird behavior.
- Why most agent failures are context failures or tool failures, not model failures.

## Outline

1. **Context** — system prompt, user prompt, files read, tool results, memory. Everything the model "sees" on this turn.
2. **Tools** — read/write/edit files, run shell, fetch URLs, search code, etc. The verbs the agent has.
3. **Loop** — model emits tool calls → harness executes → results return → model decides next step → repeat until done.
4. Diagram: a single turn vs a full task.
5. Diagnosis exercise: "the agent did something stupid" → was it context, tools, or loop?

## Key takeaway

When something goes wrong, ask the three questions in order: did it see the right things? did it have the right tools? did the loop terminate cleanly? This beats prompt-tweaking nine times out of ten.
