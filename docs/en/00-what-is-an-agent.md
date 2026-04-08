# 0. What an AI coding agent really is

Most developers met LLMs through autocomplete (Copilot) or chat (ChatGPT). An *agent* is neither. The qualitative jump is not the model — it's the **loop**: a model that can read files, run tools, observe results, and decide what to do next, repeatedly, until a task is done.

## What you'll learn

- The difference between autocomplete, chat, and agent.
- Why the loop — not the model — is what makes the difference.
- A first vocabulary: tools, context, turn, tool result.

## Outline

1. **Autocomplete** — predicts the next token. No state, no tools, no decisions.
2. **Chat** — multi-turn conversation. Still no tools or filesystem access.
3. **Agent** — model + tools + loop. Reads, edits, runs commands, observes, iterates.
4. A worked example: "fix this failing test" in each of the three modes.
5. Why this changes how you should think about reviewing AI-generated code.

## Key takeaway

Stop comparing agents to autocomplete. They're different machines. The unit of work is no longer "a suggestion you accept" — it's "a task you delegate and verify."
