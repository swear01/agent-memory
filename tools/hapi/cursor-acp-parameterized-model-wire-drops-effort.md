---
title: Cursor ACP parameterizedModelPicker 合成 wire 漏掉 effort，--model 被 Cursor 拒絕
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
updated: 2026-09-11
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

# 症狀

`hapi cursor` spawn 時 ACP 子行程 exit code 1，stderr 為
`Cannot use this model: grok-4.6[fast=false]. Available models: auto, gpt-5.3-codex-low, …, cursor-grok-4.6-high, …`，
runner log 顯示 `ACP session/new attempt 1 failed` 後重試仍失敗。引發失敗的 argv 是
`--model grok-4.6[fast=false]`，也就是 HAPI 自己的 model picker catalog 產生的值。

# 根因

HAPI 在 ACP `initialize` 帶 `clientCapabilities._meta.parameterizedModelPicker: true`
（`AcpSdkBackend.ts:196`）。帶此 capability 後 Cursor 回傳的是「裸 base + 參數 configOptions」形式，
而不是完整 parameterized wire：

- `model`（category=model）：38 個裸 base，例如 `grok-4.6`、`composer-2.5`
- `effort`（category=thought_level）：`low | medium | high | xhigh`
- `fast`（category=model_config）：`false | true`

`buildCursorModelsSnapshotFromAcp` 只認得 `model` + `fast`，對每個 base 合成
`${base}[fast=${fast}]`（`cursorAcpModelsSnapshot.ts:72`，currentModelId 在 `:103`），
完全忽略 `effort`。因此 catalog 的 76 筆 id 都是「參數不完整」的 wire。

cursor-agent 對 `--model` 的實測驗收規則：

| 傳入值 | session/new |
| --- | --- |
| `grok-4.6`（裸 base） | OK |
| `grok-4.6[effort=high,fast=false]`（完整參數） | OK |
| `grok-4.6[fast=false]`（缺 effort） | FAIL |
| `grok-4.6[effort=high]`（缺 fast） | FAIL |
| `composer-2.5[fast=false]`（該模型只有 fast 一個參數） | OK |

因此同一份 catalog 內只有 `composer-2.5[fast=*]` 能 spawn，其餘 74/76 全部被拒。
裸 base 一定可用，是因為 Cursor 允許省略參數改用預設值；一旦寫了 `[`，就必須把該模型的
參數集合寫完整。

# 為什麼清 cache 沒用

catalog 是即時生成的：刪掉 `~/.hapi/cache/cursor-models.json` 後再探測，寫回的仍是同樣
76 筆 fast-only wire（以 `runCursorAcpModelProbe` + `seedCursorModelsCache` 重跑驗證，
`availableModels` 與刪除前完全相同）。cache 只是把錯誤結果保存下來，
`listCursorModelsWhileAcpActive` 會把 disk cache 原樣再寫回，所以 mtime 一直更新但內容不變。
另一個重點：只要機器上任一個 Cursor ACP session 持有 `~/.hapi/locks/agent-acp-active`，
`isAgentAcpTransportActive()` 就為真，runner 在 cache 缺失時會直接回傳空 catalog，
不會即時重探。

HAPI 現有防護都救不了這個值：

- `remapStaleCursorModelId`：`CURSOR_LEGACY_MODEL_BASE_ALIASES` 只有 `grok-4.5 → cursor-grok-4.5`，
  `grok-4.6` 沒有 alias；且 cache 內本來就有完全相同的 `grok-4.6[fast=false]`，
  exact match 會直接回傳原值。
- `preferSpawnSafeCatalogId` / `pickBestCatalogSku`：catalog 內沒有任何非 wire 的 `grok-4.6` 列可退。
- `pickBestCatalogWire`：請求帶 `fast=false`，而 ACP 目前只宣告 `fast=true` 的 wire，
  參數相容候選為空，remap 回傳 null。
- `filterCliSkusForWireBases`：`cursor-grok-4.6-high` 的 `cursorCliSkuBaseId` 是
  `cursor-grok-4.6`，不等於 wire base `grok-4.6`，所以 `agent --list-models` 那些
  spawn-safe 的 SKU 全被濾掉，picker 只剩不可用的 raw wire。

# 與 capability 相關的對照（容易誤判）

同一台機器、同一支 cursor-agent，若 `initialize` 不帶
`_meta.parameterizedModelPicker`，`session/new` 回的是 38 筆完整 parameterized wire
（例如 `grok-4.6[effort=high,fast=true]`），且除 `gemini-2.5-flash[]` 外全部可作 `--model`。
換句話說，問題只出在 HAPI 選用的參數化合成路徑，不是 Cursor 端 catalog 壞掉。
除錯時務必先確認 client capabilities，否則同一台機器會得到兩種互斥的 catalog。

# 立即 workaround

- New Session 選 Auto：在 tiann/hapi#1819 之後會 pin CLI `auto`（`agent --model auto`），
  不再省略 `--model`、也不再套 ACP `default[]`。舊行為把 Auto 當成帳號 default，
  見 `tools/hapi/cursor-picker-auto-is-cli-auto.md`。
- 或選 `composer-2.5`：唯一 spawn-safe 的 catalog 列。
- 需要手動指定時用裸 base 或完整參數：
  `cursor-agent --model 'grok-4.6[effort=high,fast=false]' acp`。
- 不要只刪 `~/.hapi/cache/cursor-models.json`，會在下次探測後復發。
- 若要確認實際生效的模型，不要相信 picker wire 或 spawn argv，要看 ACP session 的
  model/effort/fast configOptions currentValue。

# 修正方向

1. `buildCursorModelsSnapshotFromAcp` 合成 wire 時必須帶入所有相關 configOptions
   （至少 `thought_level` 的 effort），例如 `grok-4.6[effort=high,fast=false]`；
   只合成 fast 的做法只有在該模型僅有 fast 參數時才成立。
2. `cursorModeConfig.ts:173` 的 `applyParameterizedCursorModel` 目前只 set `model` + `fast`，
   也要 set effort，否則 in-session 切換與 spawn wire 的語意不一致。
3. spawn 路徑需要安全退路：請求 wire 的參數集合不完整時退回裸 base
   （cursor-agent 接受裸 base，且 ACP 端仍可用 configOptions 套用參數）。

# 修正結果（PR tiann/hapi#1822，2026-09-11）

最後採用的是「不再合成 wire」而非「補齊參數」：每個模型可接受的參數集合是 Cursor 端的知識，
無法從 `configOptions` 推導（`composer-2.5` 只有 `fast`；`claude-opus-4-8` 有
`thinking`/`context`/`effort`/`fast`；`grok-4.6` 只有 `effort`/`fast`），所以任何合成 wire 都可能
對某些模型缺參數。改為原樣輸出 ACP 宣告的裸 base（`--model` 一定接受，預設值由 Cursor 套用），
由 in-session 的 config option 套用參數。

- `CursorModelsSnapshot`/`CursorModelsResponse` 新增 `parameterized` 旗標（model option 全為裸 base 即為真）。
  有此旗標時，variant CLI sku 仍會掛回裸 base 底下供 picker 使用，因為參數化路徑可用 config options 表達。
- `applyParameterizedCursorModel` 在 set 完 base 之後**重新讀取** `configOptions`，依序嘗試
  `effort` → `reasoning` → `reasoning_effort`（最後才 fallback 到 category `thought_level`，
  因為該 category 也包含 Claude 的 `thinking` 開關，不可誤寫），再套用 requested 的 effort；
  模型沒有該軸時直接跳過。pinned wire 只包含真正套用成功的參數。
- `~/.hapi/cache/cursor-models.json` 改為 `{version: 2, response}` envelope；舊檔直接忽略以強制重探
  （`listCursorModels` 信任 disk cache，不重探，否則舊的錯誤 catalog 會一直留存）。
- 裸 base 的 spawn 安全再驗證：36 個非 default base + 12 個抽樣 CLI sku，48/49 可 spawn。

# 實測的 effort 軸對照（cursor-agent 2026.09.10-fd3934a）

| 模型 | effort option id | 值 |
| --- | --- | --- |
| `claude-opus-4-8` / `claude-opus-5` / `grok-4.6` / `grok-4.5` / `gemini-3.6-flash` / `gemini-3.7-flash` / `muse-spark-1.3` | `effort` | `low` `medium` `high` `xhigh` `max`（grok 無 `max`） |
| `gpt-5.5` / `gpt-5.6-sol` / `gpt-5.6-terra` / `gpt-5.6-luna` / `gpt-5.4` / `gpt-5.3-codex` | `reasoning` | `none` `low` `medium` `high` `xhigh`（`gpt-5.5` 用 `extra-high` 而非 `xhigh`） |
| `gemini-3.8-flash` | `reasoning_effort` | `low` `medium` `high` |
| `kimi-k3` | `reasoning` | `low` `high` `max` |
| `glm-5.2` | `reasoning` | `high` `max` |
| `composer-2.5` / `gemini-3-flash` / `gemini-3.5-flash` / `gemini-3.1-pro` / `gpt-5-mini` / `kimi-k2.7-code` | 無 | 只有 model（部分含 `fast`） |

`thinking`（`false`/`true`）是獨立的 `thought_level` 選項，只出現在 Claude 系列與 `claude-opus-4-8`
這類模型；`context`（`272k`/`1m`/`300k`）屬 `model_config`。

# 仍未修好的 Cursor 端個案

`gemini-2.5-flash` 在**任何**形式都被 Cursor 拒絕：裸 base、`gemini-2.5-flash[]`
（Cursor 自己在無 capability catalog 中列出的完整 wire）都不行。這是 Cursor 端缺陷，
HAPI 只能照實揭露，無法修；picker 仍會顯示該列。

# 重現方式

以 JSON-RPC 對 `cursor-agent acp` 送 `initialize`（分別帶與不帶
`_meta.parameterizedModelPicker`）再送 `session/new`，比較 response 是 result 還是
`Cannot use this model`；stderr 的 `Available models:` 清單就是 Cursor 端真正接受的值。
逐筆檢查 catalog 可快速量化影響範圍（本次為 2/76 可 spawn）。
