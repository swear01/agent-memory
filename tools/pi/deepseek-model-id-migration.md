---
title: Pi fleet 的 DeepSeek model id 遷移與四層 allowlist
scope: tools/pi
project: hapi
tool: pi
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-11
tags:
  - pi
  - deepseek
  - gateway
  - model-filter
  - fleet
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

# 2026-09-11 fleet  rollout 實測結果

七台已註冊機器全部收斂到**只剩** `opencode-go/deepseek-flash` 與 `opencode-go/deepseek-pro`
（用 `GET /api/machines/:id/pi-models` 逐台讀 effective 清單）：mazu、cthulhu、athena、valkyrie
（NFS 共享 home，一次編輯四台）、zeus、oracle、Mac `swairM5`。

Gateway 實測（`POST /v1/chat/completions`，max_tokens 16，`x-opencode-session` 帶固定值）：
mazu `:35001`、zeus `:35002`、oracle `:35001` 兩個新 id 都回 HTTP 200；Mac 以新 id 開的
session 能持續推論，代表其 gateway route 生效。舊 id 的 route 仍保留在 `routing.yaml`
當 fallback（opencode-go 三個帳號當月額度用盡時，全部流量落到 command-code）。

`~/.local/bin/pi` 是 `pi-safe` wrapper（第 4 層），它對 `pi --list-models` 的 stdout 再過濾；
NFS 群與 zeus 的 wrapper 都已改成兩個新 id，oracle 與 Mac 沒有這層 wrapper。因此
`pi --list-models` 在 NFS 群現在只列這兩筆。

`swop`（Windows）在這次 rollout 無法處理，三個獨立障礙都已實測：
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
