---
title: Codex Luna Reserve — HAPI 控制邊界與條件式 Usage 顯示
scope: tools/hapi
project: hapi
tool: Codex app-server
status: active
updated: 2026-09-12
---

## 已確認的產品需求

追蹤：GitHub tiann/hapi issue #1779。官方 `v0.29.1` 已合併 PR #1780；fleet 維護版當時仍是 `v0.29.0.6`，尚未部署這段切換。真實帳號 entitlement 與 Reserve 扣額仍未驗證。

Luna Reserve 是一般額度耗盡後，部分合資格帳號可使用的 Luna 備援額度；有獨立上限。不是 Fast/Standard service tier，不得放入全域模型預設；目前採 session 條件式模型選項（見下方更新）。

使用者明確要求：正常使用時，Codex Usage **不顯示** Luna Reserve，即使後端已有未使用的 Reserve 額度。只有一般額度耗盡且官方啟用 Reserve 後才顯示；Reserve 用完時保留耗盡狀態，退出 Reserve 後再隱藏。不得把「額度存在」等同「已啟用」。

## 實際整合邊界

HAPI 不直接呼叫 OpenAI 推論 API；Codex app-server 負責推論與工具。HAPI 遠端模式自行啟動 app-server、提交 turn/start，並帶 session 模型設定，並非單純轉播官方終端介面。

2026-09-06 檢視官方 main：TUI 的 app/backend_banner_fallback.rs 透過 thread/settings/update 套用後端允許的切換；chatwidget/backend_banners.rs 決定 fallback/recovery。不可假設只升級 app-server 或加 Usage 一列就能繼承官方 TUI 的自動切換。

HAPI 遠端整合需同步有效模型，避免下一回合重新傳入舊 session 模型而撤銷切換；本地終端模式則應由官方 CLI 處理，HAPI 同步結果。恢復、resume、本地/遠端 handoff 必須重新對齊實際狀態，不重播已執行訊息。

官方 luna_reserve_model.rs 借用普通 Luna 顯示與 reasoning metadata，但保留 Reserve 請求識別碼 gpt-reserve。這是當時官方實作的協定細節；不另建公開模型、全域預設、客製推論代理或冗餘 routing 設定。實作時重新確認必要性。

## 額度與啟用不可混用

官方 main 的 account/rateLimits/read 新參數 supportsLunaReserve 宣告客戶端具備備援能力；不能在只有顯示、沒有完整接續支援時假冒此能力。

ordinaryUsageAllowed 是後端驗證目前帳號後的普通額度許可，null 是未知。不能由 usedPercent 或重設時間推斷資格或恢復。Reserve 啟用依官方後端通知/狀態，不能僅判斷普通額度百分比。

rateLimitsByLimitId 保存多組額度。百分比僅供顯示：有效 usedPercent 才轉為剩餘 100 − usedPercent；缺值未知，不當 0%/100%。週期與 reset time 依後端資料，不硬寫。更新依 limitId 合併或重讀完整快照，Reserve 的 weekly 不可覆蓋普通 weekly。

## 版本與證據限制

本機生成的 codex-cli 0.153.4 協定有 rateLimitsByLimitId、rateLimitUpsell，但沒有新 supportsLunaReserve／ordinaryUsageAllowed 欄位。官方 main 已包含不等於已發布；尚未確認最小可用 release，亦未驗證帳號 entitlement 或實際扣 Reserve 額度。

2026-09-11 Unix fleet（mazu / cthulhu / athena / valkyrie / zeus / oracle / Mac）的 live CLI 已換成 npm `@openai/codex@0.154.0`。這次只驗證 `codex --version`，沒有重跑 app-server `account/rateLimits/read`，因此不能把 0.154.0 當成已具備 Luna Reserve 協定。HAPI 的 Reserve 切換在官方 `v0.29.1`（PR #1780）；fleet HAPI 當時仍是 `0.29.0.6`。既有 session 進程不會自動換成新 Codex。

官方 `v0.29.1` 的切換是同一條 Codex thread 的 `thread/settings/update`：後端授權時寫入隱藏模型 `gpt-reserve`，HAPI 畫面顯示 `gpt-5.6-luna`，picker 不加選項。進入條件是 `ordinaryUsageAllowed === false` 且 upsell banner 為 `luna_reserve`，不是 ordinary 百分比用盡。恢復後切回進 Reserve 前記住的模型；被擋的那一輪不重送。舊 session 要新 HAPI CLI 進程 resume 同一 thread 才會走這段邏輯。

部署來源 swear01/hapi v0.29.0.5（7a89deefb）已有 Codex Usage：cli/src/codex/utils/codexUsage.ts 正規化、session.ts 合併 metadata、ComposerButtons.tsx 顯示。當時 upstream checkout 與維護分支不同；不能用 upstream 缺少此功能推斷部署版也沒有。

## 可重新核對的來源

- OpenAI Help Center article 20001499：Luna Reserve in Codex and ChatGPT Work。
- openai/codex：codex-rs/app-server-protocol/src/protocol/v2/account.rs。
- openai/codex：codex-rs/backend-client/src/client/rate_limit_resets.rs。
- openai/codex：codex-rs/tui/src/app/backend_banner_fallback.rs。
- openai/codex：codex-rs/tui/src/chatwidget/backend_banners.rs、luna_reserve_model.rs。
- swear01/hapi：cli/src/codex/codexRemoteLauncher.ts、codexAppServerClient.ts、utils/appServerConfig.ts。

## 0.153.4 真實協定驗證補充

2026-09-06 直接啟動本機 codex app-server：initialize 成功；account/rateLimits/read 帶 supportsLunaReserve:false 物件回傳 -32600，改傳 null 則成功。初次能力探測應使用 null；確認新版 ordinaryUsageAllowed 欄位與 thread/settings/update 能力後，才傳 supportsLunaReserve:true。不能只查 TypeScript 型別而假設舊 server 忽略新參數。

官方 ac192cd7937 的 Reserve fixture 使用 limitId=base_model_inference、limitName=gpt-reserve。不能假設 rateLimitsByLimitId 的 key 等於模型別名；依 limitName 找 Reserve，普通 codex bucket 保持獨立。

## 2026-09-12 手動切換方案與實測更新

PR #1780 合併後已被 upstream 撤回；新方案以現有模型選單提供條件式 `☾ Luna Reserve`，wire model 仍為 `gpt-reserve`。HAPI 不自動進入／退出、不重播失敗 turn；以 native `thread/settings/updated` 確認成功。普通 Luna 與 Reserve 顯示名稱分開，Reserve 額度只在目前使用 Reserve 時顯示；不能沿用舊版把 Reserve 隱藏成普通 Luna 的畫面。

官方 TUI 有 automatic fallback/recovery 與額度輪詢；「官方會先詢問同意且不輪詢」是錯誤推論。HAPI 的手動政策不會取消並行 native TUI 自身的行為；必須同步 native settings。HAPI 讀額度不應寫入 agent state／activity，也不需要 idle polling。

本次對已安裝 Codex 0.154.0 重新生成 protocol schema，已確認 `supportsLunaReserve` 與 `ordinaryUsageAllowed`；真實 `account/rateLimits/read` 也可解析普通額度。當時 `ordinaryUsageAllowed=true` 且沒有 Reserve banner，只能證明 HAPI 的條件式選單尚未開放；不能據此斷言原生 `gpt-reserve` 推論不可用。

CLI 全套測試使用共用 `/tmp/hapi-test-config.json`；同時跑另一個 CLI suite 會在 teardown 刪除此檔，使整批 suite 在 setup 失敗。全套與 targeted CLI 測試應依序跑，避免把測試環境競態誤判為功能回歸。


## 真實 Reserve 驗證與查詢邊界

2026-09-12 使用隔離 temporary cwd、ephemeral thread 與 `modelProvider=openai`，原生 `thread/start` 明確回覆 `model=gpt-reserve`；單次 `turn/start` 成功輸出 `RESERVE_OK`，`turn/completed` 為 completed，沒有 model/rerouted 通知。Token usage 有實際 input/output 計數。這是原生 Reserve 真實推論成功，與 mocked native response 測試不同。

真實帳號的 `model/list(includeHidden=true)` 未列出 `gpt-reserve`，但可直接建立該模型 thread。推論後 account usage 同時回傳普通 `codex` 與 `base_model_inference`（`limitName=gpt-reserve`、`normalModelSlug=gpt-5.6-luna`）週額度。一般額度未耗盡、沒有 luna_reserve banner，不代表原生模型一定拒絕請求。也不能因模型可直接推論就跳過 HAPI 既定的條件式選單政策。

新 HAPI PR #1836 已實作手動選單、獨立額度、原生通知確認、設定派送期限、自訂 collaboration instructions 保留，以及不阻塞事件投影的每回合一次提示。最新程式 head `9db722a58` 的 CI 全數通過，HAPI Bot 無 actionable issue；本機全套 7,506 passed / 7 skipped。


兩次最小真實請求均完成，每次 native token usage 為 input 49,181 / output 7。第二次請求前後，一般週額度 usedPercent 均為 13，Reserve 週額度均為 2；可確認 Reserve 模型推論及獨立 quota bucket，但百分比未變，不能宣稱本次扣額增量已被量測。不可為了製造可見百分比差異而無限重試或刻意耗盡一般額度。
