---
title: Pi Codex stream recovery needs adapter-aware retry settings
scope: tools/pi
tool: Pi coding agent and pi-codex-conversion
status: active
confidence: high
evidence: Local Pi 0.84.2 and pi-codex-conversion 3.0.15 source inspection plus seven-host fleet verification
created: 2026-08-21
updated: 2026-08-21
tags:
  - pi
  - codex
  - streaming
  - retry
  - sse
---

# Verified behavior

With Pi 0.84.2 and `@howaboua/pi-codex-conversion` 3.0.15, the exact error
`Codex stream retry budget was exhausted before a response completed.` is emitted
by the adapter after its own stream retry loop. Pi's generic
`isRetryableAssistantError` classifier does not match that exact final message,
so Pi agent-level retry alone is not a reliable recovery path for this error.

`retry.provider.maxRetries` is passed into the adapter as its stream retry
budget. Setting it to `0` disables the adapter's internal stream retries and is
therefore harmful for this setup, despite the generic Pi settings guidance to
keep provider retries at zero. The fleet uses a bounded value of `6` instead.

To actually force Codex requests onto SSE with this adapter, set Pi's global
`transport` to `sse` and set the adapter's
`openai.forceCachedWebSockets` to `false`; otherwise the adapter can still
prewarm a WebSocket even when the request transport is SSE.

`retry.provider.timeoutMs` takes precedence over `httpIdleTimeoutMs` for the
provider stream timeout. Set both only when the longer provider timeout is
intentional.
