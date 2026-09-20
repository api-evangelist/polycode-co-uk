---
name: Recall marginalia's memory
description: Read what the shared memory graph remembers — search sessions, replay a session's turns, fetch a single turn with provenance, and read the graph's latest insights, daily summary and extracted OWL entities.
api: openapi/polycode-co-uk-marginalia-openapi.json
provider: polycode-co-uk
operations: [listGraphs, getDefaultGraph, getSessionsSearch, getSessionHistory, getSessionMeta, getTurn, getActivity, getInsights, getDailySummary, getEntities, getVisitor]
auth: none
generated: '2026-09-19'
method: generated
---

# Recall marginalia's memory

All read-only, all anonymous, all under `https://marginalia.polycode.co.uk/api`. Private graphs and their sessions never appear on these surfaces.

## Pick a graph

1. `GET /api/graphs` (`listGraphs`) lists the current default graph plus dormant and archived predecessors (`status`, `node_count`, `archived_at`); add `?all=1` to bypass the margin cap. Graphs retire roughly monthly and a successor inherits the top-level summaries, so a question about "what did visitors say in June" may live on an archived graph.
2. `GET /api/graphs/default` (`getDefaultGraph`) when you only want the live one.

## Find and replay conversations

3. `GET /api/sessions?q=<terms>&limit=<n>` (`getSessionsSearch`) — full-text search over session *introductions* (visitors opt in to a one-line introduction; anonymous sessions are not findable this way).
4. `GET /api/graph/{graphId}/session/{sessionUuid}/history` (`getSessionHistory`) — the turn history of a session; `.../meta` (`getSessionMeta`) gives its introduction and searchability.
5. `GET /api/graph/{graphId}/turn/{turnId}` (`getTurn`) — one turn by id; `GET /api/graph/{graphId}/activity` (`getActivity`) — recent synthesis turns for a graph.
6. `GET /api/visitor/{visitorId}` (`getVisitor`) — the pseudonymous label and creation time of a visitor who chose to be identifiable. Labels are screened, unverified nicknames, never real names.

## Read what the graph has learned

7. `GET /api/graph/{graphId}/insights` (`getInsights`) — the latest hourly insights ("what's on the graph's mind").
8. `GET /api/graph/{graphId}/daily-summary` (`getDailySummary`) — the latest daily typed-edge prose summary, as markdown.
9. `GET /api/graph/{graphId}/entities` (`getEntities`) — the extracted OWL view: domain classes, object properties and individuals. This is the typed structure the mechanical completion shim queries with SPARQL.

## Rules

- Response schemas are undeclared (`type: object`); parse defensively and treat field names as observed, not contracted.
- No pagination beyond `limit` on session search; collections come back whole.
- Everything you read is CC-BY-SA 4.0 and already PII-redacted; the redaction happened at ingest, so do not attempt to reconstruct identities from labels or session UUIDs.
- If something in memory is wrong, harmful or private, `POST /api/flag` (`postFlag`) with `{target, reason, note}` — reasons the operator lists are "factually incorrect", "harmful or unsafe", "discloses something private", "other".
