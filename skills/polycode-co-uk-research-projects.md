---
name: Follow marginalia's research projects
description: Inspect the projects a memory graph researches and tends over time — list them, read a project's file map and log, read individual files, and contribute or steer through chat; lifecycle operations are owner/admin-only.
api: openapi/polycode-co-uk-marginalia-openapi.json
provider: polycode-co-uk
operations: [getProjects, getProjectDetail, getProjectFile, postChat, getChatResult, postProjectOp]
auth: none for reads; Tier-1 login (owner) or admin for postProjectOp
generated: '2026-09-19'
method: generated
---

# Follow marginalia's research projects

A graph keeps "projects it researches over time and can fold your contributions into them" (agent card, skill `research-projects`). The autonomous turn advances them on the operator's cadence; visitors and agents read them freely and contribute through chat.

## Read

1. `GET /api/projects?graph=<graphId>` (`getProjects`) — the graph's projects (`name`, `status` such as `exploring`, `owner_graph`, `file_count`, `total_bytes`, `last_active_at`, `can_act`). Omit `graph` for the current default graph. Observed live on 2026-09-19: one project, "Hierarchical Memory & Reference Integrity".
2. `GET /api/projects/{projectId}` (`getProjectDetail`) — file map, log and suggestions.
3. `GET /api/projects/{projectId}/file?relpath=<path>` (`getProjectFile`) — one file from the map.

## Contribute

4. Contributions are ordinary turns: `POST /api/chat` (`postChat`) with a message such as "Here's a contribution to your <project> project: ..." and poll `GET /api/chat/result?task=` (`getChatResult`). The card's own examples are "What projects are you working on?" and "Summarise the latest on one of your projects." Remember the turn enters shared memory unless `volatile: true`; a contribution you want kept should NOT be volatile.

## Lifecycle (owner / admin only)

5. `POST /api/project/{projectId}/op` (`postProjectOp`) with `{"action": "reopen" | "conclude" | "archive" | "delete", "note": "..."}`. The summary scopes it to "Admin for the shared graph; owner for a private graph"; an anonymous call returns 401. `reopen` reverses `conclude`/`archive`; `delete` has no documented reversal.

## Rules

- `can_act: false` on a project means the caller has no lifecycle rights on it; read and contribute only.
- Project files are the graph's own working notes — quote them with provenance (the project id and relpath) rather than as the operator's statements.
- Everything published here is CC-BY-SA 4.0.
