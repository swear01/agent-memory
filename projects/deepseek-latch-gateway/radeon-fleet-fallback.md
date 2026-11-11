---
title: Radeon Cloud 成為 fleet 全域 deepseek-flash 備援（priority -1）
scope: projects/deepseek-latch-gateway
project: deepseek-latch-gateway
tool: CLIProxyAPI v8.0.53
status: active
created: 2026-11-11
updated: 2026-11-11
tags: [gateway, cliproxyapi, amd, radeon, priority, failover, fleet, transfer_MAC, renderer]
---

# 決定與範圍（2026-11-11 已部署並驗證）

使用者要求 AMD Radeon Cloud 以 `priority: -1` 加到**整個 fleet**（與 NVIDIA NIM、OpenRouter 同層），
不再限定 Mac 測試機。七個 gateway 全部完成：Mac（35001）、mazu、cthulhu、valkyrie、
zeus `swear01`（`ssh zeus`，35001）、zeus `swear02`（`ssh su_zeus`，35002）、Oracle。
使用者自己聲明這些 DeepSeek 流量都是測試用途，並已知 AMD 會記錄提示詞與輸出。

- 每個 gateway 的 `<home>/.secrets` 加 `DEEPSEEK_GATEWAY_RADEON_API_KEY=rc-…`（沿用 Mac 那把），
  這樣之後重新 render 也不會掉；金鑰值不進文件或記憶。
- `cliproxy.yaml` 新增 `amd-radeon` 群組（`https://developer.amd.com.cn/radeon/api/v1`，
  `DeepSeek-V4.1-Flash` → alias `deepseek-flash`，`priority: -1`，
  `request-scoped-errors: insufficient credits → continue-and-cooldown`），
  `max-retry-credentials` 同步 7 → 8。
- transfer_MAC PR #89 已合併（`2025fe9`）：把文件改成 fleet 全域啟用、註明 192 併發上限、
  並保留「送進 Radeon 的內容對 AMD 可見」的事實描述。先前「不得在 Oracle／正式工作 Gateway 設定」
  的政策句已移除；這是使用者的決定，不是漏改。

# Radeon 免費層的實測行為

- 常態滿載：`POST /v1/chat/completions` 直接打上游會回 HTTP 429
  `model_concurrency_rate_limit_exceeded`「at its concurrency limit (192)」。
  這是共用的免費併發池，不是帳號額度用完。
- CLIProxyAPI 收到 429 會把該 credential 冷卻（`save-cooldown-status: true`，這裡約 60 秒），
  同一 request 繼續走其他 `deepseek-flash` 上游；實測 0 階上游沒額度時，
  request 會落到 Radeon 並拿到 `self-dploy/DeepSeek-V4.1-Flash` 的成功回應。
- 有值但常 429：偶爾成功，代表設定有效，不能只看一次 429 就判定路由壞掉。
- 只在 `request-scoped-errors` 有 `status: 429` 的探測中，行為與沒加時無法區分（樣本被其他
  上游額度不足污染），所以**沒有**加 429 規則，維持與 OpenRouter／NVIDIA 相同的形狀。
- 驗證陷阱：無 `x-opencode-session` 的 curl 可能被路由到 OpenCode Go 並回
  `MissingSessionID`，看起來像 gateway 壞了；測試一律帶 header。

# 部署手法（本 fleet 的實際形狀）

- 全 fleet 共用**同一份** `cliproxy.yaml`，只有 `oauth.auth-dir`（各帳號 home）與
  `server.port`（zeus swear02 = 35002）不同；金鑰直接內嵌在檔案的 `keys[].api-key`，
  不是 `${VAR}`。
- 因此部署＝改一份 JSON、逐台替換 `~/.config/deepseek-gateway/cliproxy.yaml`（`0600`，
  先留 `<file>.bak-<ts>`）、`<home>/.secrets` 補 key、再重啟：Linux
  `systemctl --user restart deepseek-gateway`，Mac
  `launchctl kickstart -k gui/$(id -u)/com.swear.deepseek-gateway`。
- 事後核對：`systemctl --user is-active`、`GET /v1/models`、一次真實 `deepseek-flash`
  completion，並確認 `amd-radeon` 在 provider 清單裡。
- Mac 上 `/v1/models` 不需要 session header，但 chat completion 要。

# 最大的坑：renderer 會整檔覆寫

`python3 scripts/sync-ai-agent-configs.py render-deepseek-gateway` 用**本機 checkout** 的
`config/deepseek-gateway/config.template.json` 重寫整份 `cliproxy.yaml`。用落後的
checkout 跑一次，就無聲刪掉較新 main 才有的上游：本次實際把 `command-code-3` 從 Mac
live 設定移除、`max-retry-credentials` 8 → 7（transfer_MAC PR #88 才加的第三個 CC 槽）。
檔案 watcher 會立刻重載，所以「只是 render 一下」等於改線上設定。

- 規則：改任何 gateway 設定前先 `git -C transfer_MAC pull --ff-only`，render 後比對
  provider 清單與 `max-retry-credentials` 再重啟；本次靠事先的 `.bak` 還原避免掉一組上游。
- 反向好處：main 更新後重新 render，Mac 產出的檔案與六台遠端部署檔（除去 auth-dir／port）
  完全一致，可直接當 fleet 的 canonical 來源。
