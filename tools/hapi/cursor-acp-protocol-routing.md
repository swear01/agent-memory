---
title: Cursor ACP and legacy session protocols must remain explicitly separated
scope: tools/hapi
tool: HAPI Cursor launcher
status: active
confidence: high
evidence: >-
  Current Cursor launcher and hub metadata tests bind cursor session IDs to
  their protocol and reject unsupported legacy/ACP resume combinations.
created: 2026-08-20
updated: 2026-09-07
tags:
  - hapi
  - cursor
  - acp
  - routing
source_refs:
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.ts
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.test.ts
  - hapi:hub/src/store/sessions.ts
  - hapi:hub/src/store/sessions.test.ts
  - hapi:hub/src/sync/syncEngineAutoMigrate.test.ts
  - hapi:shared/src/cursorCliSku.test.ts
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Routing contract

New Cursor sessions use ACP. Legacy `stream-json` sessions are not silently
loaded through ACP: an unsupported resume path fails explicitly instead of
switching protocols behind the user's back.

`cursorSessionId` and `cursorSessionProtocol` are a pair. A metadata update
that writes a new session ID must drop the old protocol; an update that omits
the pair may carry both forward so archive/resume state remains usable. A
fresh ACP session must flush the metadata that pins its ACP ID before entering
the first turn, so a hub restart cannot strand the durable session handle.

Model/configuration synchronization and permission handling may be layered on
top of ACP, but must not change the protocol-selection rule or introduce a
legacy fallback after an ACP load failure.

# Migration and wire identity

A legacy-to-ACP migration is successful only after the ACP session is ready.
If ACP loading fails, preserve the source state and surface the failure; do
not delete or merge the legacy row. Cursor CLI model labels are not ACP wire
IDs. When a base-only SKU is mapped to a wire catalog that contains fast
variants, select the explicit `fast=false` entry rather than guessing a fast
variant.

# Pending permission 與模型確認

已核對的兩個 Cursor sessions 在報告 Write permission queued 後，仍持續顯示
active=true/thinking=true；檔案停止更新不是仍在推理的證據。比對 permission
queue、最後工具事件與產物 mtime，分開報告「等工具批准」與 provider safety halt。

啟動 argv 指定 cursor-grok-4.6-medium，不代表 ACP 設定回讀確認成功。
若 log 表示 model 不在 configOptions 並 skipping，保留 requested model 與
實際 effective model/effort 的差別；此警告本身也不證明已換成另一模型或 Fast。
不要用 agent models 清單或 spawn metadata 代替 ACP runtime 確認。

# 已重現：缺少 arguments 導致待批准狀態無效

HAPI 0.29.0.6 lockfile 使用 Zod4.4.3。PermissionAdapter 的 deriveToolInput
在 rawInput/rawOutput 都無資料時回傳 undefined；JSON 傳輸會省略 arguments。
AgentStateRequestSchema 的 arguments:z.unknown() 在4.4.3要求欄位存在：
帶明確 undefined 的記憶體物件通過，JSON round-trip 後失敗，error path 指向
requests/request-id/arguments。明確 null 可 round-trip。4.2.1 對同一最小輸入
反而通過，因此必須使用 lockfile版本重現，不可只依 package range 或其他工具的Zod。

兩個報告 session 的唯讀 persisted requests 都缺 arguments；同時出現
Ignoring invalid agentState value from ack。applyVersionedAck 保留舊的 valid state
但前進 version；下一次從舊 state 建構更新會漏掉前一筆請求。DB各只剩最後一筆，
backend則仍等所有queued requests。Hub sessionCache也將invalid state映射null。
這是批准狀態傳遞錯誤，不是檔案系統拒寫，也不同於provider cyber_policy。

default mode 不會自動批准Write，文字授權不會自動轉成ACP決策。修正應在共用
pending/completed輸入邊界保留真實的缺值語義，並測JSON序列化、多請求保留、
逐筆批准與reject/cancel；不能改成全域自動允許。僅切mode也未證明能解除既有pending。
診斷證據與修正驗收：CPAchecker issue201；尚未部署修正。
