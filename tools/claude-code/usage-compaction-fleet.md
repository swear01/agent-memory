---
title: Claude Code 用量：compact 門檻、idle compaction 與 fleet 部署
scope: global
status: active
updated: 2026-10-24
---

2026-10-22 分析 Mac 近 14 天 123 個 session：99% token 是 Opus，72% 發生在 context >400K。1M 模型預設約 967K 才 auto-compact；`start_session` 用 `fork` 時會忽略 `model`、沿用 Opus。

已套用（Mac + mazu/cthulhu/valkyrie 共用的 swear01 home、zeus、oracle；athena 共用 NFS 檔案，但人在送修、未驗證）：

- `~/.claude/settings.json`：`modelSettings.claude-opus-5-5.autoCompactWindow=400000`、`claude-sonnet-5-5=200000`，`env.CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS=100000`（最低 100K）。備份檔為 `settings.json.bak-20261022`。sync-ai-agent-configs.py 只 merge hooks，不會覆寫這些鍵。
- 2026-10-24 加上 `modelSettings.claude-haiku-5-5.autoCompactWindow=100000`：Mac、mazu、cthulhu、valkyrie、zeus、oracle 已套用（各自備份 `settings.json.bak-20261024`）；athena 連不上（Network is unreachable）；swop 可 SSH，但寫入 settings.json 被 auto mode classifier 以 Self-Modification 擋下，只做了備份，尚未套用任何 compaction 設定。fleet 本來就沒有 `CLAUDE_CODE_SUBAGENT_MODEL(_FORCE)`，只有 Mac 有。Haiku 5.5 原生 1M context（不需 `[1m]`），超過 100K 的 prompt 單價較高；`autoCompactWindow` 合法範圍 100000–1000000，100K 已是下限。Context 面板的 `/ 1M` 顯示的是模型總上限，不受 `autoCompactWindow` 影響；這個設定只改 compact 門檻，不減少初始載入。
- 減少初始 context 的有效槓桿是停用用不到的 MCP server（`/mcp` 面板、`disabledMcpjsonServers`、`--strict-mcp-config`）和 plugin（`enabledPlugins`）。MCP 工具 schema 預設已 deferred（`ENABLE_TOOL_SEARCH` 未設）。
- 內建 idle compaction 只在 interactive terminal 生效；Desktop Code tab（SDK 模式）不會觸發。anthropics/claude-code#103040 尚未回應，#103189 已以 not planned 關閉。
- 本地 plugin `idle-compact@local`：原始碼在 transfer_MAC 的 `stow/agents/.agents/claude-plugins`（同時記在 `plugins.json`），Mac 直接從 repo 載入；Linux 主機用 `~/.claude/local-plugins` 的複本，改了要重新複製。邏輯在 `mods/idle-compact/hooks/register.ts`：閒置 50 分鐘且 ≥100K 就 compact；Desktop 拒絕直接呼叫時改送 `/compact`；超過 56 分鐘跳過。未裝第三方 plugin（intenex 會自動更新、1 星；xyjk0511 是 0 星）。
- Archive 會停掉 process，所以已 archive 的 session 不會被 compact；Desktop 已開「PR 合併／關閉時自動 archive」。
- AGENTS.md 規則改成 `start_session` 必帶 `model: "claude-sonnet-5-5"`、`context: "fresh"`；主 session 由使用者自選 Opus。Desktop 的 Prompt suggestions 已關閉（每個建議都是一次帶完整 context 的 request）。
- 驗證狀態：plugin 的 validate、tsc、2 個 mock-clock 測試都通過；10/14 之後的 transcript 只有 manual 和 auto 兩種 compact，沒有 idle。plugin 還沒在真實 session 看到觸發過，若閒置回來沒出現 `/compact` 就要查。
- Windows swop：2026-10-22 SSH 兩次 timeout；2026-10-24 SSH 可連，但 settings.json 寫入被 classifier 擋，仍未部署。
- 「快到 5 小時上限就收工」用 handoff，不要 compact。Desktop 用 `get_usage` 讀 5h%，CLI statusline 讀 `rate_limits`；hook 讀不到用量（#38380）。
