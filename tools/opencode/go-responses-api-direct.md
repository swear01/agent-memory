---
title: 直接呼叫 opencode Go Responses API（GPT-6 Luna）的注意事項
scope: tools/opencode
tool: opencode Go
status: active
created: 2026-10-22
updated: 2026-10-22
tags: [opencode-go, gpt-6-luna, responses-api, cloudflare, rate-limit]
---

# 直接呼叫 opencode Go Responses API

2026-10-22 實測 `POST https://opencode.ai/zen/go/v1/responses`（`gpt-6-luna`、`reasoning.effort=low`）：

- 前面有 Cloudflare。Python `urllib` 預設 User-Agent `Python-urllib/3.x` 會得到 HTTP 403、body `error code: 1010`；送任意明確的 `User-Agent` 即 200。curl 預設 UA 可通過。
- 必須帶 `x-opencode-session`；可用 `instructions` 放 system prompt、`input` 放使用者內容。回應 `reasoning.effort` 會回顯 `low`，`usage.output_tokens_details.reasoning_tokens` 有回報。
- 額度用完回 HTTP 429，body `{"type":"error","error":{"type":"GoUsageLimitError","message":"Go usage limit exceeded"},"metadata":{"limitName":"weekly"}}`，並帶 `Retry-After` 秒數（當天是 321631 秒，到 2026-10-26 08:00 台北時間）。除了 5 小時滾動額度，還有**週額度**；遇到 `limitName: weekly` 時固定間隔重試沒有意義，應依 `Retry-After` 停下並回報。
- 研究實驗時，key 只在程式內讀取；request/response log 不含 `Authorization`。

詳見 [NeuroAbs decision 實驗](../../projects/rtl-relate/neuroabs-decision-experiment-2026-10-22.md)。
