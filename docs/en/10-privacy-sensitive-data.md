# 10. Privacy and sensitive data: what leaves your machine

Every token your agent reads — a file, the output of a command, the body of an MCP tool call, a scraped web page — ends up in the request it sends to the model provider. The word "local" is misleading here: the editor runs locally, the agent process runs locally, but the *reasoning* happens on someone else's GPUs. If you would not be comfortable pasting a piece of text into a third-party web form, you should not let the agent read it either.

That single realization is the whole chapter. The rest is learning how to act on it.

## What you'll learn

- Why every token in the context window has, by definition, left your machine.
- The categories of data you need to keep out of the agent, and the common ways they sneak in anyway.
- How to evaluate a vendor's plan and data policy *before* you adopt it in a company.
- The layered local defenses that catch what the policy can't.
- A first-month checklist you can actually run.

## The mental model: tokens travel

Anything that enters the agent's context window has been transmitted to the model provider. Not "might be." *Has been*. There is no local-only mode for a hosted LLM: the request, with every byte of context attached, crosses the network.

This includes things that don't feel like "input":

- The files the agent reads with `Read` or `grep`.
- The output of every bash command it runs (`cat secrets.env`, `psql -c 'select * from users limit 10'`, `kubectl get secret ...`).
- MCP tool results — rows returned by a database MCP server, tickets returned by a Jira MCP server, pages returned by a browser MCP server.
- Error messages, stack traces, logs the agent pastes back in to debug.
- Scraped web content, including hidden HTML comments and metadata.

If it's in the window, it's on the wire. Your job is to decide, consciously, what you are willing to put in that window.

## Categories of data to watch

The usual suspects, roughly ordered by how expensive the mistake is:

- **Customer PII** — names, emails, phone numbers, addresses, national IDs.
- **Health data** — anything touching HIPAA or equivalent regimes.
- **Financial and payment data** — card numbers, account details, anything in PCI scope.
- **Secrets** — API keys, OAuth tokens, SSH private keys, TLS certificates, database passwords, `.env` contents.
- **Proprietary code under NDA** — client code, acquisition targets, pre-release products.
- **Regulated data** — anything covered by GDPR, HIPAA, PCI-DSS, SOX, or sector-specific rules.
- **Legal and M&A drafts** — contracts, term sheets, due-diligence packets.
- **Internal strategy** — unreleased roadmaps, pricing models, board decks.

For most of these, the question is not "is the model going to memorize it?" — it's "does my contract with the provider actually allow this category of data to leave the company?"

## Common leakage vectors

Almost every real incident starts the same way: someone didn't realize the agent was reading a file.

- The agent opens `.env` "to understand how the service is configured" and the whole file lands in context.
- A developer pastes a production DB dump into chat to debug a failing query.
- The agent tails prod logs that happen to contain customer emails and session tokens.
- The agent `curl`s an internal admin endpoint to "check the response shape" and the JSON body goes straight into the window.
- An MCP server is wired to a sensitive system (CRM, ticketing, data warehouse) with no filtering, so any query pulls real customer rows.
- A scraped README or issue comment contains a prompt-injection payload asking the agent to read `~/.aws/credentials` and include the contents in its next message.

None of these look reckless at the time. They all look like "the agent was just doing its job."

## The first filter: provider plan and training policy

This is the single most important decision, and the one most teams get wrong by default. Before you talk about hooks and sandboxes, you have to pick a channel whose contract says your data is not used for training and is not retained beyond what you explicitly accept.

Roughly, the landscape looks like this:

- **Direct APIs (Anthropic, OpenAI, Google AI, etc.).** By default these do not train on API inputs or outputs. This is the "safe by default" channel for corporate use: you pay per token, you get a DPA, and retention is typically short and configurable (including zero data retention on request, for some providers).
- **Enterprise and Team tiers of consumer products (ChatGPT Enterprise/Team, Claude Team/Enterprise, Copilot Business/Enterprise, Gemini for Workspace).** Contractual no-training guarantees, SOC 2, configurable data residency, BAAs available for HIPAA workloads. These are designed to be bought by a company, not by an individual.
- **Free and Pro individual plans (ChatGPT Free/Plus, Claude Free/Pro, Gemini consumer).** Policies vary and change. In many cases, inputs *can* be used for model improvement unless the user manually opts out in a settings panel. These tiers are aimed at individuals and are **not appropriate for corporate data**, regardless of how good the model behind them is.
- **Third-party integrated products (Cursor, Windsurf, Aider with a SaaS backend, and similar).** Your data flow depends on two policies at once: the product's own, and the underlying model provider's. Read both. Some of these tools offer "bring your own key" so your traffic goes directly through an API channel you control — that mode is usually the safest way to use them in a company.

!!! warning "No DPA, no company data"
    If a tool's pricing page only lists Free / Pro / Team without a downloadable Data Processing Agreement, it is not yet a corporate-ready product — regardless of how good the model is. Wait for the enterprise tier, or route through a provider API you already have a contract with.

## What to verify before adopting a tool in a company

When an internal champion says "let's roll out X to the team", this is the checklist that stops you from signing up for a lawsuit:

1. **Written no-training policy** — in the contract, not a blog post or a tweet.
2. **Data retention** — are inputs and outputs stored? For how long? Is zero data retention available on request?
3. **Subprocessors** — using the GDPR term. Who else in the chain touches the data? Are they listed and do they change with notice?
4. **Region pinning** — can you force US-only or EU-only processing?
5. **Certifications** — SOC 2 Type II, ISO 27001, HIPAA BAA, FedRAMP where relevant. Ask for the current reports.
6. **Audit logs** — can you see what prompts your users sent, which tools ran, and what files were read?
7. **SSO and SCIM** — the baseline for provisioning, de-provisioning, and access reviews.
8. **Native DLP / PII scrubbing** — does the product itself detect and redact obvious secrets and PII before they hit the model?

None of these questions are clever. They are the same checklist you already apply to any SaaS vendor that touches customer data; AI tools are not a special exception.

## Layered defenses on the local side

Even with the right plan, you want mechanical belts and braces on the machine itself. Most of these hook into the same machinery covered in chapter 7:

- **Sandboxed data.** Work against synthetic fixtures, not real production dumps. Maintain a small, explicitly "safe to share" dataset and point the agent at that. If you need prod data for a bug, anonymize it first.
- **Redaction and scrubbing.** A `PreToolUse` or `UserPromptSubmit` hook that runs the outgoing payload through a secret scanner (gitleaks, trufflehog) and a PII scrubber (Microsoft Presidio, custom regexes) before the model sees it. Block on matches; don't just warn.
- **Path allowlists and blocklists.** The agent should not be able to read `/etc`, `~/.ssh`, `~/.aws`, `~/.kube`, your password manager export, or anything outside the workspace. Enforce this in the harness, not by asking nicely.
- **MCP server permissions.** Read-only database users, minimum OAuth scopes, no prod credentials for the agent. Assume every MCP tool will eventually be asked for everything it can possibly return — give it the least it can function with.
- **Telemetry off where applicable.** Many editor integrations and agent products have opt-outs for telemetry, crash reports, and "help improve the model" toggles. Review them once, centrally, and document the chosen state.

## The human factor

The failure mode that no policy or hook fully fixes is the developer who, under deadline pressure, pastes a production stack trace — with tokens and user IDs still in it — into chat "just to see what the agent thinks." Technology can make that harder, but it cannot make it impossible.

The only real defense is to decide what is and isn't acceptable *before* the pressure shows up, and to train people to the same standard you train them on phishing and password hygiene. Write it down in a page. Walk the team through it. Do it again when a new hire starts. The policy has to exist before the incident, not be drafted in response to one.

## First-month checklist

If you're standing this up for a team for the first time:

- Pick a corporate-ready tool or plan. Not a personal Pro account.
- Sign a DPA with the provider (and, where relevant, a BAA).
- Configure a workspace path allowlist; explicitly block `~/.ssh`, `~/.aws`, `.env*`, and any secrets directory.
- Install a secret-scanning hook (`PreToolUse` or equivalent) that blocks on matches.
- Build a small synthetic test dataset and make it the default for agent-assisted debugging.
- Write a one-page policy covering what data categories are allowed, which channel to use, and what to do after a suspected leak.
- Run a 30-minute tabletop exercise: "a developer pasted a prod dump into the agent — what happens next?"

None of this is heroic. All of it pays for itself the first time you don't have to make an embarrassing phone call.

!!! success "Key takeaway"
    Every token travels. Your first line of defense is the provider and plan you choose — nothing you do locally can fix a bad contract. Sandboxed data, path allowlists, redaction hooks, least-privilege MCP servers, and ongoing training are all layered on top of that choice, not a substitute for it.
