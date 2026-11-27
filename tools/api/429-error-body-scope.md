---
title: "429 診斷需保留錯誤內文與驗證範圍"
scope: "tools/api"
status: active
updated: 2026-11-28
evidence_digest: ae746da94c7c71e10d6009cabfaf25e2a92f8c012ba2c4cf1b16657e802ffae0
---

429 或 `RESOURCE_EXHAUSTED` 時，從原始 response body、請求端點及模型確認是速度限制、額度還是預付餘額耗盡。歷史案例只有 embedding 的 response 明示 `prepayment credits are depleted`。解除阻擋後，分別重試 embedding 與 completion 並讀回結果。

Provider 拒絕參數時，要對照當前端點和 client 的 request schema。相容端點可能要求在 `create` 頂層傳 `reasoning_effort`，同名設定未必放在 `extra_body`。用實際請求驗證接受位置和允許值。
