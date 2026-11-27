---
title: Cursor ACP Cannot use this model：picker 合成了不完整參數
scope: tools/hapi
tool: HAPI Cursor ACP launcher
status: fixed-in-pr
confidence: high
evidence: >-
  在 swever 以 cursor-agent 2026.09.10-fd3934a 對 HAPI 0.29.0.6 產生的 76 筆
  catalog 逐筆做 initialize + session/new，只有 2 筆 spawn-safe；並用 hapi cli
  的 runCursorAcpModelProbe + seedCursorModelsCache 重現同一份 catalog。修正後再以
  真實 ACP session 驗證 36 個 base + 12 個抽樣 sku 可 spawn、effort option 逐模型
  對照，見 tools/hapi/cursor-acp-parameterized-model-wire-drops-effort.md 後段。
created: 2026-09-11
updated: 2026-11-28
tags:
  - hapi
  - cursor
  - acp
  - model-picker
  - thought_level
source_refs:
  - hapi:cli/src/cursor/utils/cursorAcpModelsSnapshot.ts
  - hapi:cli/src/agent/backends/acp/AcpSdkBackend.ts
  - hapi:cli/src/cursor/utils/cursorModeConfig.ts
  - hapi:cli/src/cursor/utils/cursorStaleModelRemap.ts
  - hapi:shared/src/cursorCliSku.ts
  - hapi:cli/src/modules/common/cursorModelsSharedCache.ts
  - hapi:cli/src/modules/common/cursorModels.ts
related:
  - tools/hapi/cursor-acp-protocol-routing.md
redaction: passed
---

看到 `Cannot use this model: grok-4.6[fast=false]` 或 `ACP session/new attempt 1 failed`，先查看實際 spawn argv 與 ACP init 的 `clientCapabilities._meta.parameterizedModelPicker`。同一版 Cursor 帶 capability 會回傳裸 base 和分開的 `configOptions`，不帶則可能回完整 wire。舊 HAPI 將裸 base 與 fast 合成 ID，漏掉該模型必要的 effort。清 cache 後仍會重建同一錯誤 ID；ACP 活躍鎖存在時，缺 cache 還可能直接得到空 catalog。

- 舊版 workaround 是選 `auto`（在 [#1819](https://github.com/tiann/hapi/pull/1819) 後會傳真正的 CLI Auto），或裸 base。以當時 Cursor 實測，`grok-4.6` 與 `grok-4.6[effort=high,fast=false]` 可用，fast-only 或 effort-only 會失敗。只有 fast 軸的 `composer-2.5[fast=false]` 可用。
- [#1822](https://github.com/tiann/hapi/pull/1822) 改為送原生裸 base，再套用 in-session options。先 set base 並重新讀 `configOptions`，依 `effort`、`reasoning`、`reasoning_effort` 比對，最後才依 category，避免把 `thinking` 開關當 effort。Cache v2 忽略舊 envelope。Pinned wire 只記真正 ACK 的選項。
- 用同二進位對 `initialize` 的兩種 capability 做 `session/new` 對照，再讀目標 session 的 model／effort／fast `currentValue`。picker、argv 與 catalog 都不是實際值。當時 `gemini-2.5-flash` 兩種 catalog 形式都被 Cursor 拒絕；重測安裝版本再判斷。

[整理前完整紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/tools/hapi/cursor-acp-parameterized-model-wire-drops-effort.md)
