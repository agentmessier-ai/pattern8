# Pattern8 — Claude Code plugin

Search **battle-tested, layer-classified production patterns** from real open-source
projects, straight from Claude Code. Build on proven implementations instead of
reinventing — and when the catalog is missing something, report the gap and it is
reviewed for research.

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
  - `pattern_get(pattern_name, full_code?)` — **exact match**, keyed on a pattern's own
    unique name. The full reference: contract → implementation → resilience → quality
    attributes → code. Set `full_code=true` when you are about to implement it: it
    appends the complete source of every function the mechanism uses, each chunk headed
    with its file, line range and upstream commit (large — request it only for the
    pattern you will build from).
  - `pattern_feedback(query, feedback)` — report that the catalog had no good answer.
    The gap is recorded for review by a human curator; valuable gaps are researched and
    the resulting pattern lands in a later catalog release. It is **not** researched
    instantly, so there is nothing to poll for right after submitting. State the gap
    concretely (what is missing, under what constraints) — specifics sharpen the result.
  - `pattern_job_status(job_id?)` — status of research the team has run on a gap you
    reported. With no argument, lists your completed, unread results. Most feedback has
    no job to poll; when research for you does finish, it also surfaces automatically
    on your next tool call.
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
