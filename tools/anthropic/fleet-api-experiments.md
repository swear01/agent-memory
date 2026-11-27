---
title: Claude API 實驗的憑證範圍、workspace header 與讀回
scope: tools/anthropic
status: active
updated: 2026-11-11
tags: [anthropic, fleet, credentials]
---

## 路徑與授權

- 個人批次實驗使用明確選定的個人 API 憑證，勿走訂閱帳號的 `claude -p`，也不要依賴繼承的 `ANTHROPIC_API_KEY` 或 shell 預設 fallback。只在當次有授權的實驗使用 lab key。2026-11-11 的 Agentic-IC3 Opus 補跑允許個人額度耗盡或 rate limit 時用 lab key 接續，此條只描述該案。
- 互動式 Claude Code 走[訂閱啟動入口](claude-subscription-launcher.md)。wrapper 只清理該子程序的認證環境，直接 HTTP／SDK 保持獨立。
- 歷史設定使用 `ANTHROPIC_API_KEY_SWEAR` 與 `ANTHROPIC_API_KEY_LAB`。個人 org key 當時不帶 workspace header，lab identity-linked key 需送 `anthropic-workspace-id`，否則模型清單可回 HTTP 400。`anthropic-use-lab` 設 workspace，`anthropic-use-swear` 清除它。Python SDK 要明確提供 header，不能由 CLI 環境變數行為推定。
- SDK 不會自動改用另一把 key。舊 shell／子程序會持有舊環境，新 shell 或明確傳入授權的憑證後重驗。使用 vault 或既有受控環境，只讀回結果與所用身份，避免列印值、shell 設定、請求 header 或把值貼進終端命令。

## 驗證界線

`GET /v1/models?limit=1` 非空 200 只說明可連、認證及 workspace 通過該端點，生成要用另一次經授權的實際呼叫驗證。NFS 共用 home 可更新一次，各 host 的實際 caller 獨立讀回。舊記錄在各日期涵蓋不同機器，不能由 HAPI 在線或設定存在推定遺漏的機器曾完成驗證。

[原始診斷與部署快照](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/anthropic/fleet-api-experiments.md)
