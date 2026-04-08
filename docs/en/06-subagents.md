# 6. Subagents and delegation

A subagent is a fresh agent the main agent launches to handle a sub-task in isolation. Two reasons to use them: **parallelization** (independent searches at once) and **context protection** (keep the main thread clean from large noisy outputs).

## What you'll learn

- The two real reasons to delegate to a subagent.
- When to do the work inline in the main thread instead.
- How to brief a subagent so it actually returns something useful.
- The single failure mode that ruins delegation: outsourcing your own thinking.

## The two real reasons

### 1. Context protection

The main agent's conversation is precious. Every token spent on a 4,000-line `grep` output is a token that's not spent on the actual problem. A subagent can run that `grep`, read the matches, and return *one paragraph* of synthesis. The noise stays in the subagent's context and dies with it.

### 2. Parallelization

Three independent questions — "where is auth handled?", "what does the user model look like?", "how are emails sent?" — can be answered by three subagents at the same time. Doing them sequentially in the main thread is just slower for no benefit.

!!! note "Both reasons are about the main context"
    Either you're protecting it from noise, or you're filling it faster by parallelizing. If neither applies, you don't need a subagent.

## The anti-reason

!!! warning "Don't delegate because it sounds fancy"
    Delegating "rename this variable" to a subagent is slower, more expensive, and harder to debug than just doing it. The cost of spinning up a subagent — briefing, waiting, parsing the response — is real. Pay it only when the savings are real.

## Inline vs delegate: a cheat sheet

| Situation | Do it inline | Delegate to subagent |
|---|---|---|
| Edit a file you're already looking at | Yes | No |
| Search 200 files for a pattern | No | Yes — return only matches |
| Read three unrelated files in parallel | No | Yes — one subagent each |
| Decide which approach to take | Yes | No — never delegate the decision |
| Summarize a 2,000-line log file | No | Yes — return the summary |
| Write the final code change | Yes | No — you own the synthesis |

The pattern: **delegate the gathering, keep the deciding.**

## Briefing a subagent

A subagent doesn't share your conversation. It walks in cold. Brief it the way you'd brief a colleague who just sat down at your desk: goal, context, constraints, expected output.

### A concrete example prompt

Suppose the main task is "add rate limiting to the public API." Before writing code, you want to know how the existing middleware stack is organized. Here's a good subagent brief:

```text
Goal: Map the HTTP middleware stack of this Spring Boot service so I can
decide where to insert a rate-limiter.

Context:
- Repo root: /Users/me/work/payments-api
- Framework: Spring Boot 3.3, Java 21
- I am about to add a rate limiter for the /v1/public/** routes
- I have NOT yet decided between Bucket4j and Resilience4j

Please investigate:
1. List every class annotated with @Configuration that registers a
   filter, interceptor, or WebMvcConfigurer customization.
2. For each, note the order (Ordered / @Order) and which URL patterns
   it applies to.
3. Identify the current authentication filter and where in the chain
   it sits.

Constraints:
- Read-only. Do not modify any files.
- Do not recommend a library — I'll decide that myself.
- Budget: ~5 minutes of exploration.

Expected output:
- A table: filter/interceptor name -> order -> URL pattern -> file path
- A 3-sentence summary of where a new filter would naturally fit
- Nothing else
```

Notice the structure: a single goal, the context the subagent needs (and only that), explicit constraints (read-only, no library recommendation), and a precise output format. The subagent will return ~30 useful lines instead of 3,000 noisy ones.

!!! tip "If you can't write the brief, you're not ready to delegate"
    A vague brief produces a vague answer. If you struggle to specify the expected output, the task probably isn't well-defined enough to delegate yet.

## Parallel delegation

When sub-tasks are independent, fire them in parallel in a single turn. A typical "explore an unfamiliar codebase" turn might launch three subagents at once:

- One mapping the HTTP layer.
- One mapping the persistence layer.
- One mapping the background jobs.

Each returns a summary; the main agent stitches them together. The whole exploration finishes in the time of the slowest subagent, not the sum.

## Specialized subagents

Some teams define named subagents for common roles: a *research* subagent (read-only, web + code search), a *planner* (produces a plan, writes no code), a *code-search* subagent (greps, reads, returns paths and snippets). Specialization is useful when the role is reused often enough to justify a stable brief — otherwise, an ad-hoc subagent with a good prompt is fine.

## The fatal failure mode

!!! warning "Never delegate the synthesis step"
    The moment you ask a subagent "...and then decide which approach we should take and start implementing it," you've handed away the most important part of the job. The subagent doesn't have your full context, doesn't know the trade-offs you've already weighed, and doesn't carry the responsibility for the result. The synthesis — *what does all this mean and what should we do?* — is yours. Always.

## Key takeaway

!!! success "Key takeaway"
    Subagents are for protecting context and parallelizing — not for offloading thinking. Delegate the gathering, keep the deciding, and never delegate the synthesis step.
