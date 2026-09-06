---
title: ACP permission arguments must survive the JSON ACK boundary
scope: tools/hapi
status: active
updated: 2026-09-06
---

HAPI issue #1781：ACP 的 rawInput 與 rawOutput 都缺少時，permission entry 產生 arguments: undefined。JSON 會移除這個 key；鎖定的 Zod 4.4.3 對 arguments: z.unknown() 要求 key 存在，故 pending/completed agent state 失效。

ApiSessionClient 的 versioned ACK 仍推進版本，但忽略無效 agent state。下一筆更新從先前有效狀態建立，導致前面 pending requests 被覆蓋。只用 handler(state) 的記憶體假物件無法覆蓋此問題；測試必須讓 outgoing state 與 ACK 經過 JSON round-trip，再用 AgentStateSchema 驗證。

2026-09-06 本地驗證：PermissionAdapter、AcpPermissionHandler、CopilotPermissionHandler、GrokPermissionHandler、OpencodePermissionHandler 五個橋接器都有相同缺值問題。將重複的 deriveToolInput 合併到 cli/src/agent/utils.ts，使用 rawInput !== undefined ? rawInput : rawOutput ?? null，可保留明確 null、false、0、空字串，以及 rawOutput fallback，並讓缺值用 JSON-stable null 表示。缺值測試由五個失敗轉為通過。

驗證範圍：真實 CLI client、RPC handler、版本 ACK 與 schema，模擬 Socket.IO transport；並非真實 Cursor、Hub storage/SSE 或瀏覽器 E2E。不得把此根因當成其他 agent 所有批准卡住問題的通用診斷，也不應以自動批准繞過狀態損壞。
