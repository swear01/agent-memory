---
title: Pi web-search credential 空值、shell 展開與設定回讀
scope: tools/pi
status: active
confidence: high
created: 2026-08-26
updated: 2026-08-30
tags: [pi, pi-web-access, web-search, fleet]
---

以 `playground/docs/web-search-toolkit-plan.md` 及現行安裝的 extension 核對版本與能力。2026-08-26 的實測為 `pi-web-access` 0.25.0，8/30 已有 0.27.0 部署紀錄。不要從這些歷史 rollout 推定目前安裝、可用 provider 或帳戶額度；[Pi 安裝核對](../hapi/fleet-codex-pi-install.md) 說明如何讀回實際本機入口。

## 可重現的設定坑

- `~/.local/bin/pi-websearch-apply` 從 `~/.pi/web-search.json.template` render `~/.pi/web-search.json`。只為已設定且非空的憑證寫入引用，用時才讀取憑證。共享 home 只 render 一次，各 runner 使用的設定與實際搜尋要各自讀回。
- credential command 的內層 shell 必須收到原樣的變數引用。用雙引號包 `sh -c` 的命令可能使外層 shell 先展開 `${X:-}` 成空；內層引用要用單引號。若 helper 使用 `${!k}`，它需要 Bash，`dash` 不支援。
- `command-empty`／`environment-empty`：0.25.0 的空 credential 會直接失敗，不會回退 keyless Exa MCP。刪掉無值引用，別以空佔位值保留 provider；驗證只記解析狀態，不顯示值。
- Firecrawl 在 0.25.0 沒有預設 base URL，template 當時需明確設定 `firecrawlBaseUrl`。專用 provider 是否參與 auto/all 順序和收費，重試前看當前設定與指標。402 不能只憑歷史費額判定。
- 若原設定已有空的 placeholder 行，append-only 合併可能跳過真正應填的項目；應由目前的憑證來源安全更新後重新產生設定，避免印出完整設定或憑證。

[原始設定、提供商快照及驗證](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/pi/web-search-toolkit.md)
