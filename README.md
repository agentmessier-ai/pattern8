# Pattern8 — Claude Code plugin

Search **battle-tested, layer-classified production patterns** from real open-source
projects, straight from Claude Code. Build on proven implementations instead of
reinventing — and when the catalog is missing something, Pattern8 researches it for you.

## What it adds

- **MCP server `pattern8`** with 5 tools, each named for its mechanism and key:
  - `pattern_search(query, language?, constraints?)` — **fuzzy search.** Describe what
    you need in plain language; returns matching library choices (install command,
    rationale, code example, alternatives) plus implementation patterns via hybrid
    search — no exact name required. `constraints` narrows results by compatibility
    axes (e.g. `{"infra_deps": ["none", "db-only"]}`).
  - `pattern_by_capability(capability, layer?, language?)` — **exact match**, keyed on
    a capability name (e.g. `rate-limit`, `auth`) — no fuzzy matching, use when you
    already know the exact capability.
  - `pattern_get(pattern_name)` — **exact match**, keyed on a pattern's own unique
    name. The full reference: contract → implementation → resilience → quality
    attributes → code.
  - `pattern_feedback(query, feedback)` — report a gap; queues live background
    research to fill it. A preliminary report (which repos it's checking, and why) typically
    lands in about 5 minutes; extraction results follow as each repo finishes and are
    delivered automatically on your next tool call.
  - `pattern_job_status(job_id?)` — check research queued via `pattern_feedback`.
    With no argument, lists your unread completed jobs — useful at the start of a
    session to pick up research that finished after you last asked.
- **Skill `use-pattern8`** — nudges Claude to consult the catalog before implementing any
  production concern, and shows how to compile a user's request into an effective query.

## Install

In Claude Code:

```
/plugin marketplace add agentmessier-ai/pattern8
/plugin install pattern8@pattern8
```

The plugin connects to the hosted `pattern8` MCP server at
`https://pattern8.agentmessier.ai/mcp/`. The first tool call triggers a one-time
OAuth login (GitHub or Google) in your browser — no API key to manage. Once
authorized, the session persists across restarts.

## Plans

The free tier includes full-quality results (code, evidence, citations) — sign in on
first use and you're on it automatically. Current quotas, rate limits, and paid plans:
[pattern8.agentmessier.ai/pricing](https://pattern8.agentmessier.ai/pricing).

## How it works

Pattern8 is a thin client. The catalog and its search run as a hosted service;
this plugin just connects Claude Code to its MCP endpoint via OAuth 2.1.

## Layers

| Layer | What | Examples |
|---|---|---|
| technical | cross-cutting infra | input-validation, rate-limit, http, cache, auth |
| business | domain-agnostic mechanisms | event log, audit, state machine, reconciliation, saga |
| domain | industry-specific | ledger, clinical-records, cart/checkout, billing |
