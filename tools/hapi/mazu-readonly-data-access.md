---
title: HAPI hub data is readable through a namespace-scoped read-only API
scope: tools/hapi
project: hapi
tool: hapi
status: active
confidence: high
evidence: HAPI source review plus a read-only local hub smoke test.
created: 2026-08-19
updated: 2026-08-19
tags:
  - hapi
  - hub
  - sessions
  - export
  - api
  - redaction
source_refs:
  - public-release:hapi-readonly-data-access
  - hapi-source:hub/src/web/routes/auth.ts
  - hapi-source:hub/src/web/routes/sessions.ts
  - hapi-source:hub/src/web/routes/messages.ts
  - hapi-source:shared/src/sessionExport.ts
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Access boundary

The HAPI hub exposes namespace-scoped session data through its authenticated web API. The preferred bulk-ingestion path is the hub's loopback HTTP endpoint, reached directly on the hub host or through an SSH tunnel. A public tunnel or Cloudflare edge may apply an additional access policy and is not the reliable data plane.

Exchange the configured `CLI_API_TOKEN` for a short-lived JWT:

```http
POST /api/auth
Content-Type: application/json

{"accessToken":"<CLI_API_TOKEN>"}
```

Use the returned JWT only in an `Authorization: Bearer <JWT>` header. Never put it in logs, URLs, shell history, or exported provenance.

The access token can select a namespace with the supported namespace form. `GET /api/sessions` returns only sessions in the JWT namespace; do not attempt to enumerate other namespaces.

# Read-only session retrieval

- `GET /api/sessions` returns namespace-scoped session summaries.
- `GET /api/sessions/:id` returns one session's metadata.
- `GET /api/sessions/:id/export` returns the session and visible messages in chronological order.
- `GET /api/sessions/:id/messages?limit=<n>&beforeAt=<timestamp>&beforeSeq=<seq>` returns a paginated message window.

The export endpoint has a 20,000 visible-message limit. When a session exceeds it, the endpoint returns `413` with the count and limit rather than silently truncating. Use the paginated messages endpoint for large sessions.

# Safe ingestion procedure

1. Query session summaries and select closed/inactive sessions first.
2. Fetch message pages with stable `(beforeAt, beforeSeq)` cursors.
3. Record page boundaries and verify that pages cover the complete source without gaps or repeated cursor progress.
4. Redact each complete message before handing overlapping chunks to subagents.
5. Keep only durable lessons and redacted evidence in Markdown; never store the JWT or raw API payload.

Prefer the API over opening the live SQLite database. If database access is unavoidable, take a quiescent local backup, open it read-only, and treat WAL/SHM files as one snapshot. Never read or write the live database from shared storage.

# Logging guard

Do not use an unfiltered service-status command as an export method. Supervisor or tunnel logs can contain bearer tokens in SSE query URLs. Capture only sanitized status codes, counts, and service state; redact query strings before retaining any diagnostic output.
