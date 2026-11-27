---
title: Daily Research selection timeout and gateway session affinity
scope: projects/routine
status: active
updated: 2026-11-27
---

- A successful gateway `GET /v1/models` confirms the tunnel and model listing. Check the matching `POST /v1/chat/completions` on the gateway to diagnose live inference.
- For the selection failure on 2026-11-27, the keyword request returned 200. The next request reused that provider through CLIProxyAPI session affinity and the gateway recorded 499 at the client's 180-second deadline. This establishes where the request stopped. A 499 gives no upstream error response. In the GitHub log, route messages printed before the tunnel step can be unit-test fixtures.
- Retry only the timed-out paper-selection inference, once, before publishing. Send a fresh UUID in both affinity headers so CLIProxyAPI can select a provider again. Keep the new session for report generation after a successful retry. A whole-job retry after publishing can repeat Discord delivery or the seen-list update.
