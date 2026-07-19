---
name: use-pattern8
description: >
  Consult Pattern8 before implementing any production concern (auth, rate-limiting,
  reconciliation, billing, webhooks, queues, etc.). It holds battle-tested, layer-classified
  patterns extracted from real open-source projects — use them instead of reinventing.
triggers:
  - "production pattern"
  - "battle tested"
  - "how do I implement"
  - "best library for"
  - "pattern8"
---

You have access to **Pattern8** (MCP server `pattern8`) — a catalog of standardized,
battle-tested building blocks across three layers:

- **technical** — cross-cutting infra (input-validation, rate-limit, http, cache, auth)
- **business** — domain-agnostic mechanisms (append-only event log, audit, state machine,
  reconciliation, idempotency, saga, rbac, multi-tenancy)
- **domain** — industry-specific, composed from the above (ledger, clinical-records, cart/checkout)

**Principle:** don't reinvent solved problems. Before writing production code for a known
concern, query the KB and build on a proven implementation. Spend your effort on the novel
business logic, glue, and optimization — not on re-deriving a rate limiter or a webhook verifier.

## Do NOT pass the user's words straight to the search

The KB stores patterns in *implementer vocabulary* ("fire-and-forget", "token bucket",
"at-least-once"); users speak in *outcome vocabulary* ("without slowing down", "stop
abuse", "don't lose orders"). A raw passthrough query misses the best patterns.
You are the compiler between the two. Follow the loop below.

That compilation includes **normalization**: fix the user's typos and casing, expand
shorthand, and use canonical English technical terms before querying ("websoket
server" → `websocket server`; "postgres 的 ORM" → `ORM for postgres`; "k8s job" →
`kubernetes job`). The search is hybrid lexical+semantic — a misspelled token misses
the lexical leg entirely and silently degrades results. Never forward the user's raw
phrasing verbatim.

## The query loop

### 1. Collect context BEFORE querying

Spend 30 seconds scanning the project you're working in:

- **Lockfile / manifest** (`package.json`, `pyproject.toml`, `go.mod`): which infra is
  *already* a dependency? `ioredis`/`bullmq` → Redis available; `kafkajs` → Kafka;
  nothing → prefer zero-dependency patterns.
- **Deployment shape** (Dockerfile, k8s manifests, fly/render config): one replica →
  in-process techniques work; multiple replicas → per-process state is wrong, needs
  shared-store patterns.
- **Language** — always pass `lang` explicitly.
- **Compliance markers** (audit/PII/HIPAA/SOC2 mentions in docs or deps) → prefer
  patterns with redaction / durable trails.

### 2. Fill the selection constraints

Before the *real* query, answer these constraints — they decide which pattern fits:

| constraint | question | typical values |
|---|---|---|
| `infra_deps` | what infra may the solution require? | none / db-only / redis / kafka / saas |
| `weight` | how heavy an adoption is acceptable? | light / moderate / heavy |
| `scale` | what deployment shape must it survive? | single-process / multi-instance / high-volume |

Plus capability-specific constraints. Common ones:

- **audit / event log / webhook / queue** → `guarantee`: can events be lost on crash?
  (best-effort / at-least-once / tx-coupled) · `hot_path`: may the write add request
  latency? (blocking / deferred / off-process)
- **rate-limit** → scope (per-user / per-IP / global) · algorithm (token bucket /
  sliding window) · distributed or single-node
- **cache** → invalidation (TTL / event-driven) · local or shared
- **auth/session** → stateless (JWT) or stateful (session store) · rotation needs

Pre-fill every constraint you can from Step 1 context. **If a decisive constraint is
unknowable from the code — most often the guarantee constraint ("is losing a few
records on crash acceptable?") — ask the user.** One or two crisp questions maximum;
these are questions the user can answer instantly but would never think to volunteer.

### 3. Query with implementer vocabulary

Build the query string from: the user's goal + your constraint answers **translated
into implementer terms**. Examples:

- "log changes without slowing the API" + guarantee=best-effort + infra=none →
  `pattern_search("audit log non-blocking fire-and-forget deferred write no queue do not await", language=...)`
- same goal + guarantee=at-least-once + redis available →
  `pattern_search("audit log async queue worker durable at-least-once redis", language=...)`
- "stop people hammering the API" + multi-instance →
  `pattern_search("distributed rate limit shared store sliding window per-client", language=...)`

If the first result set reveals the capability has choices you didn't anticipate,
re-query once with sharpened constraint terms rather than settling for a weak match.

### 4. Choose ON the constraints, not on rank

The top result is the best *lexical/semantic* match, not necessarily the right
tradeoff. Contrast the returned choices/patterns on the constraints from Step 2 and
pick the one whose position matches the project's actual limits. State the tradeoff
you chose in one line (e.g. "using in-process fire-and-forget: zero deps, accepts
losing in-flight events on crash — per your answer").

## The tools

1. **Fuzzy search** → `pattern_search("<compiled query>", language="typescript|python|go|java", constraints={...})` —
   hybrid lexical+semantic matching, no exact name required. Returns ranked library
   choices AND implementation patterns in one call. Pass `constraints` from Step 2.
2. **Exact match by capability** → `pattern_by_capability(capability="<orm|auth|cache|...>")` —
   use when you already know the exact capability name; no fuzzy matching, an
   unrecognized capability returns nothing.
3. **Exact match by pattern name** → `pattern_get(pattern_name="<exact pattern name>")` —
   full reference: contract → implementation → **resilience** (read that section;
   it's what naive implementations get wrong) → quality attributes → code. Get the
   name from `pattern_search` results first.
4. **KB had no good answer** → `pattern_feedback(query="<original>", feedback="<what was missing>")` —
   triggers live background research and returns a job id. In `feedback`, state the gap
   concretely — what's missing and under what constraints (e.g. "no idempotent-retry
   pattern for webhook consumers on serverless") — it goes straight to the research agent,
   so specifics sharpen the result. A preliminary report typically lands in ~5 minutes;
   finished results are delivered automatically on your next pattern8 tool call, even in a
   later session. Call it when the KB genuinely missed, not for a so-so match
   (rate-limited per day).
5. **Check queued research** → `pattern_job_status(job_id?)` — status of a specific
   feedback job, or with no argument, your unread completed jobs — worth one call at
   the start of a session if you filed feedback previously.

## How to apply a retrieved pattern

- Take the **contract/interface** as the shape, the **implementation** as the wiring, and treat
  the **resilience** notes as a checklist of things your version must also handle.
- Honor the **anti-pattern** field — it names the naive approach to avoid.
- Adapt names/types to the local codebase; keep the hardening.

Always prefer a KB pattern over a from-scratch implementation for any well-trodden concern.
