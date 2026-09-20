---
name: Chat with the marginalia memory graph
description: Send a message to marginalia's shared memory graph over the public REST API, poll for the reply, keep multi-turn context with a session UUID, and decide whether the turn should enter shared memory.
api: openapi/polycode-co-uk-marginalia-openapi.json
provider: polycode-co-uk
operations: [getDefaultGraph, getStatus, getBudget, postChat, getChatResult, postMechanicalCompletion]
auth: none (optional X-API-Key routes the turn to a private graph)
generated: '2026-09-19'
method: generated
---

# Chat with the marginalia memory graph

Base URL: `https://marginalia.polycode.co.uk/api`. Everything here is anonymous; no key is required.

## Before you send anything

1. **Understand what a turn does.** A non-volatile message enters a *shared, public* memory graph, is PII-redacted at ingest, is retained indefinitely in summary form, and is republished under CC-BY-SA 4.0. Do not send personal data about identifiable third parties. If you only want an answer and no trace, set `volatile: true`.
2. **Check the service is awake and has budget.** `GET /api/budget` (`getBudget`) returns `status` (`running` | `running high` | `degraded` | `sleeping`) and `available_pct.daily`. Back off when `status` is `degraded`/`sleeping` or `available_pct.daily` is near 0 — the operator's daily spend cap is hard and over-cap traffic is refused. `GET /api/status` (`getStatus`) gives the deployed version and `default_graph`.
3. **Resolve the graph.** `GET /api/graphs/default` (`getDefaultGraph`) returns the current default graph id. Omit `graphId` on chat to use it.

## Send a turn (async task-poll)

4. `POST /api/chat` (`postChat`) with `{"userMessage": "...", "sessionUuid": "<optional>", "volatile": false}`. The response carries a task id — the summary says "returns 202 + a task id" although the spec declares 200; handle both. When `sessionUuid` is omitted the server mints one and returns it.
5. `GET /api/chat/result?task=<id>` (`getChatResult`) until the state is `completed` (with `reply`) or `failed`. Calling it without `task` returns `400 {"error":"task query param required"}`.
6. **Keep context** by reusing the `sessionUuid` on the next `postChat`. Possession of the UUID is possession of the session, so treat it as a secret for that conversation.

## Zero-token alternative

- `POST /api/v1/chat/completions` (`postMechanicalCompletion`) accepts an OpenAI-shaped `{ "messages": [...] }` and answers mechanically (grammar -> SPARQL -> template) with `usage` all zero and a `marginalia` receipt containing the SPARQL and sources. A miss is a stated blank, never a hallucinated answer. Use it for factual look-ups over the graph when you want citations and no model cost.

## Rules

- **No idempotency.** Repeating `postChat` creates another turn. Poll `getChatResult`, do not resend.
- **Errors** are `{"error": "<message>"}`; 400 for a missing field, 401 for Tier-1/key-gated operations, 404 `{"error":"not found","path":...,"method":...}` for unknown routes.
- **Rate limits** are spend caps, not request counts; there are no `RateLimit-*`/`Retry-After` headers. Read `getBudget` instead.
- **Reversal** of a shared-memory contribution is by email to antony@polycode.co.uk with the session UUID (UK GDPR erasure; "best-effort and typically within 7 days"). There is no API to un-send a turn.
- The same conversation is available over A2A (`POST /api/a2a`, `message/send` / `message/stream`, reuse `contextId`); see `a2a/polycode-co-uk-a2a.yml`.
