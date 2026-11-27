---
title: Claude Code 用量：compact 門檻、idle compaction 與 fleet 部署
scope: global
status: active
updated: 2026-10-24
---

## Context 上限、auto-compact 與 idle compaction

- `~/.claude/settings.json` 的 `modelSettings.<model>.autoCompactWindow` 控制 compact 門檻，不會減少初始 context 或改變面板的模型上限。2026-10 採 Opus 5.5 400K、Sonnet 5.5 200K、Haiku 5.5 100K 與環境 `CLAUDE_CODE_IDLE_COMPACT_MIN_TOKENS=100000`。100K 當時為合法下限，核對現行安裝及模型後再沿用。設定 renderer 只合併 hooks。
- 減少初始 context 時，先查不需要的 MCP／plugin，使用 `/mcp`、`disabledMcpjsonServers`、`--strict-mcp-config` 或 `enabledPlugins`。當時工具 schema 可 deferred，單純計已載入工具數不能核定接下來的用量。
- 內建 idle compaction 只在 interactive terminal 生效；Desktop Code 使用 SDK 路徑。自有 `idle-compact@local` 的來源在 transfer_MAC `stow/agents/.agents/claude-plugins`，Mac 從 repo 載入，Linux 從 `~/.claude/local-plugins` 複本載入。程式在 `mods/idle-compact/hooks/register.ts`，閒置 50 分鐘且 ≥100K compact、超過 56 分鐘略過。Desktop 拒絕直接 compact 時 fallback 到 `/compact`。測過 mock-clock，未觀察真實 idle 觸發。Windows 部署只改 settings，沒有 plugin。
- Archive 後已無存活程序可 compact。接近五小時使用上限由 handoff 收工；Desktop 用 `get_usage`，CLI statusline 讀 `rate_limits`。當時 hook 讀不到使用量。
- 獨立 Desktop `start_session` 的 fork 可能忽略指定模型、沿用 parent。按使用者偏好選 Sonnet 與 fresh context；獨立 session 和 subagent 設定分見 [model fallback](subagent-model-force.md)。

當時 fleet readbacks、備份與用量觀察見 `git show 9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e:tools/claude-code/usage-compaction-fleet.md`。
