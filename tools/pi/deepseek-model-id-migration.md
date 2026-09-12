---
title: Pi fleet 的 DeepSeek model id 遷移與四層 allowlist
scope: tools/pi
project: hapi
tool: pi
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-12
tags:
  - pi
  - deepseek
  - gateway
  - model-filter
  - fleet
  - swop
  - todo
---

# 為什麼要改

DeepSeek 2026-09-10 發布 V4.1-Flash，官方 id 改為 `deepseek-flash`（原生 multimodal）；`deepseek-v4-flash`
與 `deepseek-v4-flash-vision-exp` 只是「temporarily routed」的相容別名，之後會移除。2026-09-14 04:00 UTC
起 `deepseek-v4-pro` 的請求也會被導到 V4.1 Flash 並改以 Flash 價計費。因此 fleet 統一改用
`deepseek-flash` 與 `deepseek-pro` 兩個 id。

Fleet 的 opencode-go / command-code 上游目錄並不一致：opencode-go(zen) 同時列出 `deepseek-flash`、
`deepseek-v4-flash`、`deepseek-v4.1-flash`、`deepseek-v4-pro`；command-code 只有
`deepseek/deepseek-v4-flash`、`deepseek/deepseek-v4-flash-fast`、`deepseek/deepseek-v4.1-flash`、
`deepseek/deepseek-v4-pro`、`deepseek/deepseek-v4-flash-vision-exp`，**沒有** `deepseek/deepseek-flash`
或 `deepseek/deepseek-pro`。所以 gateway 端必須用 `upstream_model` 做名稱映射，不能只靠 passthrough。

# Pi 的模型限制是四層，不是三層

設定分散在四個地方，只改其中一兩層會出現「檔案改了但 picker 沒變」的假象：

1. `~/.pi/agent/settings.json` 的 `enabledModels`（同時是 default provider/model 來源）
2. `~/.pi/agent/model-filter.json`（`pi-model-filter`，`defaultAction: block` 的白名單）
3. `~/.pi/agent/extensions/strict-model-allowlist.ts`（runtime prototype patch，實際擋 API call）
4. `~/.local/bin/pi` 這類 wrapper（`pi-safe`）：它對 `pi --list-models` 的 stdout 再做一次 awk 過濾，
   有自己一份硬編碼 id 清單

第 4 層最容易被漏掉，因為 `pi --list-models` 顯示的是 wrapper 過濾後的結果，而
HAPI `/api/machines/:id/pi-models` 與 pi 內部 registry 走的是前 3 層；兩者可以同時給出不同答案。
排查方式：先 `readlink -f $(command -v pi)` 找 wrapper，再另開一個 extension dump
`ctx.modelRegistry.getAll()/getAvailable()` 看真實 registry 內容。

若要在 `opencode-go` provider 下新增自訂 model id，必須在 `models.json` 的該 provider 加
`models: [{id, ...}]`；opencode-go 雖然是 pi 內建 provider，但 vendor id 來自 `models-store.json`
或 static catalog，只改 `enabledModels` 不會生出新 model。

# 已驗證的遷移內容

- gateway `config.yaml` 的 `models.allow` 新增兩個新 id（舊 id 保留，讓既有 session 續跑）。
- gateway `routing.yaml` 對每個 model 定義 priority route：`deepseek-flash` →
  command-code `upstream_model: deepseek/deepseek-v4.1-flash`；`deepseek-pro` →
  `deepseek/deepseek-v4-pro`。`deepseek-flash` 的 priority 1 仍是三個 opencode-go 帳號。
- 驗證必須實際打一次 `/v1/chat/completions`：回應 body 的 `model` 欄位會顯示上游真實 id
  （例如請求 `deepseek-flash` 得到 `"model":"deepseek/deepseek-v4.1-flash"`），這是名稱映射生效的證據。
  只 restart service 或只看 `/health` 不足以證明新 route 已註冊；log 會出現
  `[Routing] deepseek-flash: 1:opencode-go -> 2:command-code-fallback`。
- Gateway 設定只在 process 啟動時讀取，因此每台機器的 gateway 必須各自 restart；NFS 共享 home 時
  檔案只需寫一次。

# 舊 session 與 resume 的影響

allowlist 收斂後，明確指定舊 id（例如 `pi --model opencode-go/deepseek-v4-flash`）會被
`strict-model-allowlist` 擋下並拋 `Model "..." is blocked by strict-model-allowlist`；
但**已經在跑**的 session 不受影響（它已持有 model 物件）。Resume 舊 session 時 model 會
resolve 不到，於是 fallback：有 `currentModel` 就用它，否則挑預設或第一個可用 model。
這是預期行為，不是故障。

# 共享 home 的邊界

mazu / athena 共用 `192.168.1.200:/volume1/nfs-home`，cthulhu / valkyrie 共用
`192.168.1.199:/volume1/nfs-home`（同一台 NAS 的兩個介面），所以 `~/.pi/agent/*` 與
`~/.config/deepseek-gateway/*` 的 inode 相同，一次編輯即覆蓋四台；但 gateway 是每台各一個 process，
restart 必須逐台做。zeus（`<swear02-home>`，port 35002）、oracle（`<ubuntu-home>`）、Mac
（`<mac-home>`）、Windows `swop`（`<windows-home>`）各有自己的 home，要分別處理。

# 驗證路徑

HAPI hub 的 `GET /api/machines/:id/pi-models` 可直接讀出每台機器的 effective pi 模型清單，
是 fleet 級驗證最快的方式（需先 `POST /api/auth` 用 `CLI_API_TOKEN` 換 JWT）；`swop` 這台若
runner 離線會回 `RPC handler not registered` 或 500，不能用它推論設定未套用。

使用者要的是「**只把 DeepSeek 的 id 改名，其餘模型（Codex/ChatGPT、Meta、Qwen）全部保留**」。
2026-09-11 的 rollout 把 allowlist 錯誤地砍到只剩兩個 DeepSeek id，2026-09-12 已在 `swever` 修正。
正確的 Pi 端白名單是 8 個 model：`opencode-go/deepseek-flash`、`opencode-go/deepseek-pro`、
`openai-codex/gpt-5.6-luna`、`openai-codex/gpt-daybreak-blue-latest`、`openai-codex/gpt-5.6-sol`、
`openai-codex/gpt-5.6-terra`、`valkyrie-ninfer/qwen3.8-27b`，外加 `includes` 特例
`meta/muse-spark-1.2-contributor`（在 `enabledModels`／`model-filter.json` 內是實體條目）。

`includes` 的 meta 特例是關鍵：`allowed` Set 只有 7 筆，`meta/muse-spark-1.2-contributor` 靠
`includes` 裡的 `model.provider === "meta"` 判斷放行，所以 `model-filter.json` 也必須有 meta 的
allow 條目，否則 `defaultAction: block` 會把它擋掉。

## 還原時要挑對備份（2026-09-12 實證）

`~/.config/deepseek-gateway/backup-<timestamp>-pre-deepseek-rename/` 裡的 `agent__*.json`
**不是** rename 前的內容：它是 rename 之後、白名單被砍之後才複製的，`enabledModels` 與 `rules`
已只剩兩個 DeepSeek id，拿它還原會把錯誤狀態再蓋一次。真正 pre-rename 的權威副本在 pi 目錄內：

- `~/.pi/agent/settings.json.backup-deepseek-rename-20260911-113603`
- `~/.pi/agent/model-filter.json.backup-deepseek-rename-20260911-113603`
- `~/.pi/agent/extensions/strict-model-allowlist.ts.backup-deepseek-rename-20260911-113603`

還原方式就是拿這三份做 `sed -e 's|deepseek-v4-pro|deepseek-pro|g' -e 's|deepseek-v4-flash|deepseek-flash|g'`
後覆蓋，不做其他改動。pre-rename 的預設是 `defaultProvider: valkyrie-ninfer`、
`defaultModel: qwen3.8-27b`、`defaultThinkingLevel: high`（rename 順手把預設改成 opencode-go，
那不算「改名結果」，要一併還原）。

# 這一版 pi 的實際過濾行為（0.85.x 實測）

- `pi --list-models` 走的是 CLI 的 listing path，呼叫 `modelRuntime.getAvailable()`；
  `strict-model-allowlist.ts` 只在 `session_start` hook 內補這個方法，而 `--list-models` 不觸發
  session，所以**這條路徑的過濾在該 extension 內是不生效的**，真正把它收起來的是 wrapper（第 4 層）。
- `model-filter.json` 需要 `pi-model-filter` package 才有意義；沒有安裝時它就只是死設定。
- `settings.json.enabledModels` 只影響 pi 自己的預設／循環選擇，不會過濾 catalog。
- 因此「picker 只剩兩個」的證據必須來自 HAPI 的 `pi-models` API（它會經過 session pipeline），
  不能只用 `pi --list-models` 的輸出，兩者可以同時不一致。

`pi --list-models` 在完全沒有 wrapper 的機器上會列出數百個 model（oracle 實測 462 筆）。
要在 fleet 統一，就得逐台確認有沒有 wrapper，並更新它那份硬編碼清單。

# 已知未完成項：Windows `swop`

2026-09-11 無法處理，三個已實測的原因同時存在：

1. `swop` 沒出現在 hub 的 `GET /api/machines`（只有七台），`spawn-peer` 回
   `RPC handler not registered: <id>:spawn-happy-session`，代表該 runner 目前未註冊。
2. mazu → `192.168.1.206:22` 只回 `Permission denied (publickey,password,keyboard-interactive)`；
   Windows 端只授權 Mac 的 key。
3. 唯一持有該 key 的 Mac 當時在 `10.49.59.0/24`（外部／熱點網路），並非 `192.168.1.0/24`；
   `known_hosts` 裡只有 `192.168.1.206` 這個私有位址，`~/.ssh/config` 沒有 Host alias、
   沒有 ProxyJump，也沒有任何 public WAN 位址或轉埠。Oracle `swever` 本身是 `10.0.0.0/24` 的
   NAT VM，TCP/22 直接 timeout。

也就是說：要完成 swop，必須等 Mac 回到 `192.168.1.0/24`（或該 runner 重新註冊到 hub），
不是靠猜測 credentials 或改 sshd 能解決。swop 的 gateway 由 SYSTEM Scheduled Task 管理，
config/routing 位於 `<program-data>/DeepSeekGateway`，pi 設定位於 `<windows-user-home>`。

<<<<<<< Updated upstream
## PENDING TODO：swop 補做步驟（2026-09-11 使用者指示「先記著，之後弄」）

解除條件（任一成立即可動工）：Mac 回到 `192.168.1.0/24` 或開 VPN；或 swop runner 重新註冊到 hub；
或拿到 Windows 可用的 SSH alias／帳號（或把 mazu 的 key 加進 `administrators_authorized_keys`）。

動工方式：從 Mac 開一個 pi session（`hapi spawn-peer --machine <swairM5-id> --dir /Users/swear`）遠端執行；
或 swop runner 回來後直接對 swop spawn。要改的內容與 Mac/Linux 相同：

1. 備份到同目錄 `backup-<timestamp>-pre-deepseek-rename/`。
2. `%USERPROFILE%\.pi\agent\models.json`：`providers["opencode-go"]` 加 `models` 兩筆（`deepseek-flash`／
   `deepseek-pro`，input `["text","image"]`、contextWindow 1000000、maxTokens 384000、
   compat 含 `thinkingFormat: deepseek`），baseUrl/apiKey 不動。
3. `settings.json`：default `opencode-go/deepseek-flash`、`enabledModels` 只放兩個新 id、thinking `high`。
4. `model-filter.json`：只留 opencode-go 兩個新 id、`defaultAction: block`。
5. `extensions\strict-model-allowlist.ts`：`allowed` 只留兩個新 id，`includes` 移除 meta 特例。
6. `opencode.jsonc`（若有）：model／small_model／whitelist 改新 id；`.dsh\settings.yaml` 加 models 兩筆。
7. gateway `config.yaml`：`models.allow` 同時留新舊四個 id；`routing.yaml` 加 `deepseek-flash`
   （priority 1 = opencode-go-1/2/3，priority 2 = command-code `upstream_model: deepseek/deepseek-v4.1-flash`）
   與 `deepseek-pro`（priority 1 = command-code `upstream_model: deepseek/deepseek-v4-pro`）。
8. 用 `Start-ScheduledTask -TaskName "DeepSeek Gateway (SWOP)"` 重啟（不要改成臨時使用者程序）。
9. 驗證（swop 不在 hub，**不能**用 `GET /api/machines/:id/pi-models`）：在 Windows 本機 dump
   `ctx.modelRegistry.getAll()`，應只有 `opencode-go/deepseek-flash`、`opencode-go/deepseek-pro`；
   並各打一次 `/v1/chat/completions` 確認 HTTP 200。

注意：swop 的 HAPI runner 未註冊本身是獨立問題（runner 可能已停），補做時一併確認
`HAPI Runner (SWOP)` Scheduled Task 狀態，但**不要**在未經同意的情況下改動它的排程 action。

# 2026-09-11 fleet  rollout 實測結果
=======
# 2026-09-11 fleet rollout 實測結果（其中白名單收斂是錯的）
>>>>>>> Stashed changes

2026-09-11 七台已註冊機器（mazu、cthulhu、athena、valkyrie（NFS 共享 home，一次編輯四台）、
zeus、oracle、Mac `swairM5`）的 effective 清單被收斂到只剩兩個 DeepSeek id。這是**過度收斂的
錯誤狀態**，不是使用者要的結果；2026-09-12 在 `swever` 已還原成 8 個 model（Codex 4 個 ＋ meta
＋ valkyrie-ninfer ＋ 2 個 DeepSeek）。其餘機器若還停在兩個 id，比照上面的備份還原流程處理。

Gateway 實測（`POST /v1/chat/completions`，max_tokens 16，`x-opencode-session` 帶固定值）：
mazu `:35001`、zeus `:35002`、oracle `:35001` 兩個新 id 都回 HTTP 200；Mac 以新 id 開的
session 能持續推論，代表其 gateway route 生效。舊 id 的 route 仍保留在 `routing.yaml`
當 fallback（opencode-go 三個帳號當月額度用盡時，全部流量落到 command-code）。

`~/.local/bin/pi` 在 NFS 群、zeus、Mac 是 `pi-safe` wrapper（Mac 是 `pi` 實體就是 symlink；
oracle 的 `pi` 是直接指向 pi 的 `cli.js`，wrapper 只在 `.bashrc` 以 alias 生效），它對
`pi --list-models` 的 stdout 再過濾。這四組的 wrapper 都已改成兩個新 id（Mac 與 oracle 由後續
session 補做），所以互動式 `pi --list-models` 現在只列這兩筆；但直接呼叫未過濾的 pi binary
（`command pi`、絕對路徑）仍會列出全部 catalog。

`swop`（Windows）在這次 rollout 無法處理，三個獨立障礙都已實測（詳見上文「已知未完成項」）：
1. HAPI hub `GET /api/machines` 只有七台，**沒有** swop → 該 runner 目前未註冊，不能用 hub RPC 讀寫。
2. mazu 對 LAN `192.168.1.206`（OpenSSH 10.3、開 22 與 5900）SSH 只回
   `Permission denied (publickey,password,keyboard-interactive)`；Windows 只授權 Mac 的 key。
3. 唯一有 key 的 Mac 當時在 `10.49.59.0/24`（非家中 LAN），且 `~/.ssh/config` 找不到任何
   Windows/swop alias。
要在該機套用同一組改名，需先恢復 Windows 的 HAPI runner（讓 hub 能 spawn），或提供 Mac 上
可用的 Windows SSH alias／位址。

# 舊 id 的兩個保留理由

- Gateway `models.allow` 與 `routing.yaml` 保留舊 id，讓既有 session 在 allowlist 收斂前後都能續跑，
  也讓舊 id 的既有 `pi`／opencode／zed／goose 設定在改名過程中不會突然 404。
- Pi 端則刻意**不**保留舊 id（allowlist 只放兩個新 id）：明確指定舊 id 會直接被擋，避免
  「檔案改了但實際還在用被 retire 的模型」這種無聲狀態。

# 2026-09-11 後續：cost、swear-review、文件來源

- `models.json` 的 `deepseek-flash` cost 已全 fleet 改成官方 V4.1 Flash 價 `0.15 / 0.60 / 0.003`（input / output / cache hit，per 1M tokens）。NFS 四台共用一份直接改；zeus、oracle、Mac 由各自 session 改，都留了 `models.json.bak-costfix`。`deepseek-pro` 的 `0.66 / 1.98 / 0.022` 未變。
- oracle `/opt/swear-review/data/config.yaml` 的 model 由 `deepseek-v4-flash` 改成 `deepseek-flash`（baseURL 不變，仍是本機 gateway `http://127.0.0.1:35001/v1/chat/completions`），service 重啟後 log 顯示 `"model":"deepseek-flash"`。重啟中斷了一個 in-flight job（殘留 `/tmp/swear-review/job-904/repo` 已清）。Mac 端備份副本 `<mac-home>/Documents/swear-review/.e2e/config.yaml` 也已同步同一值。
- `transfer_MAC` 的文件與設定來源（`docs/mac/40_AI_Agent_MCP_and_Skills.md`、`docs/notes.md`、`docs/oracle/swear-review-remote.md`、`config/snapshot/home/.pi/agent/{settings.json,model-filter.json,extensions/strict-model-allowlist.ts}`、`stow/config/.config/opencode/opencode.jsonc`）已更新：PR https://github.com/swear01/transfer_MAC/pull/46（分支 `docs/deepseek-model-ids`）。合併後 Mac 要 `git pull`；若 Mac working tree 對 `stow/config/.config/opencode/opencode.jsonc` 已有未 commit 的相同修改（改名當時是直接改 symlink 的實體檔），pull 前先 `git checkout -- <該檔>`。
- 刻意**不**改的 id：`stow/core/.zshrc`（走 `api.deepseek.com/anthropic`）、`config/snapshot/home/.config/goose/config.yaml` 的 custom_deepseek、`stow/config/.config/zed/settings.json` 的 deepseek official 與 `open_router` slug（`deepseek/deepseek-v4-flash` 等）。那些是官方／OpenRouter 端點的 model 名稱，本機發明的 `deepseek-pro` 在那裡無效；本機 gateway 仍保留舊 id route，這些 client 不會斷。

# 2026-09-12 PR #46 合併與 Mac 同步

- PR #46 已 merge（merge commit `f0453d4`）。CI 在最新 head `0ac6630` 全綠：`Secret and code scan` pass、`Swear Review` pass（0 findings，log 顯示 `Model: deepseek-flash` — 順帶證明剛改完的 swear-review 設定可用）。Gemini Code Assist 對 opencode.jsonc 的 `deepseek-flash` 留了一條 high-priority finding（聲稱 V4.1 Flash 非推理模型、帶 `thinking`/`reasoning_effort` 會 400）；實測兩個參數打 gateway 都回 HTTP 200 且回應含 `reasoning_content`，已在該 thread 回覆反證並 resolve。
- Mac 的 `/Users/swear/Documents/transfer_MAC` 當時 `main` 是 ahead 1 / behind 3：先前 session 把 opencode.jsonc 的改名直接 commit 成 `fb843dc`。處理方式：以 origin/main 版本覆蓋該檔（`6b5bf73`）→ `git merge --no-ff origin/main`（`29a8c00`，無衝突）→ 推回 `main`。使用者自己的未提交變更 ` M stow/core/.zshrc` 全程未動。
- 遺留物：mazu 上的 task worktree `~/.agent-worktrees/transfer_MAC-deepseek-docs-20260911`（branch `docs/deepseek-model-ids`，內容已全部 merge 進 origin/main）因 `.skillshare/skills` submodule scaffold 殘留，`git worktree remove` 需要 `-f` 才能刪；依「不用 force 清 worktree」原則保留未刪，下次清理時需人工判斷。
