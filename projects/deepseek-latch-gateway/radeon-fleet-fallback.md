---
title: CLIProxyAPI 的 NVIDIA NIM／Radeon 備援、affinity 和 renderer 覆寫
scope: projects/deepseek-latch-gateway
project: deepseek-latch-gateway
tool: CLIProxyAPI v8.0.53
status: active
created: 2026-11-11
updated: 2026-11-11
tags: [gateway, cliproxyapi, amd, radeon, priority, failover, fleet, transfer_MAC, renderer]
---

- **落後 checkout render 後線上少一組 provider。** `python3 scripts/sync-ai-agent-configs.py render-deepseek-gateway` 用本機 checkout 的 `config/deepseek-gateway/config.template.json` 整份覆寫 `cliproxy.yaml`，watcher 會立即 reload。先 `git -C transfer_MAC pull --ff-only` 並以自己修改前備份作對照，核 provider 群組、`max-retry-credentials`、auth-dir／port。舊 checkout 曾把第三個 Command Code 槽和其 retry count 一起復原成舊形狀。
- **priority -1 的備援未觸發，或 400 沒有換上游。** NVIDIA NIM、Radeon Cloud、OpenRouter 依目前 template 設定為低優先 Flash 備援；不要推定 NIM 有 Pro。Established session affinity（也可能是 prompt-prefix hash）可覆蓋排序，用新 session 和新 prompt 驗證。CLIProxyAPI 預設 400 不 failover，僅針對 `insufficient credits` 加 `request-scoped-errors` 的 `continue-and-cooldown`；測試帶 `x-opencode-session`，不把 MissingSessionID 當 provider 掛點。
- **Radeon 429 或 NIM 免費層超時就判故障。** Radeon 的 `model_concurrency_rate_limit_exceeded` 是上游共享併發池而非帳號額度；已有部分真實成功，按回應冷卻及後續供應商讀回。NIM 首 token 有長延遲，同把 key 多台並測曾 500；`save-cooldown-status` 在共享 auth 目錄可讓其他主機一起冷卻。不要添加不經證實的 blanket 429 分類。Radeon 上游可見並記錄傳給它的提示與回應，沿使用者核准的流量範圍使用。
- **同一份設定或 /models 已通過就報全 fleet 新路由上線。** 逐程序核 supervisor、合法權限、enabled provider、真實 completion 的 active endpoint/attempt；不同 Zeus instance 及共用 NFS 遠端要各核。帶 key 的 runtime config 僅私人 0600。可從輸出成功證明當次有接入，不能推定所有 upstream 有容量，未在線的 host 應另記。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/deepseek-latch-gateway/radeon-fleet-fallback.md) · [NVIDIA 舊版試跑、錯誤分類與歷史價格](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/deepseek-latch-gateway/nvidia-nim-fallback.md)
