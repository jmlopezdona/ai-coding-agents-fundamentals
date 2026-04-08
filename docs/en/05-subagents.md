# 5. Subagents and delegation

A subagent is a fresh agent the main agent launches to handle a sub-task in isolation. Two reasons to use them: **parallelization** (independent searches at once) and **context protection** (keep the main thread clean from large noisy outputs).

## What you'll learn

- When to delegate vs when to do it yourself in the main thread.
- How subagents protect the main context window.
- The cost: a subagent doesn't share your conversation, so briefing matters.

## Outline

1. The two real reasons to use subagents.
2. Anti-reason: "it sounds fancy." Don't delegate trivial work.
3. Briefing a subagent like a colleague who just walked in: goal, context, constraints, expected output.
4. Parallel delegation: multiple independent subagents in one turn.
5. Specialized subagents (research, planning, code search) and when each pays off.
6. Failure mode: delegating *understanding* — making the subagent decide what you should have decided.

## Key takeaway

Subagents are for protecting context and parallelizing — not for offloading thinking. Never delegate the synthesis step.
