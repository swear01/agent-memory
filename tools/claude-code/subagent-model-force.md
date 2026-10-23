---
title: Claude Code 強制 subagent 用 Sonnet（Opus 主 agent）
scope: tools/claude-code
tags: [subagent, model, settings, claude-desktop, env]
status: active
updated: 2026-10-24
---

目標：主 agent 維持 Opus，所有 subagent 固定跑 Sonnet，省 Opus 額度且不必進 Plan Mode。2026-10-24 起 Mac 已拿掉 FORCE，見下方「改回」。

## 設定

`~/.claude/settings.json`：

```json
"env": {
  "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet",
  "CLAUDE_CODE_SUBAGENT_MODEL_FORCE": "1"
}
```

`CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` 會忽略 agent 定義的 `model:` 和 spawn 時指定的 model。只寫 agent frontmatter 的 `model: sonnet` 不可靠，曾有 regression 讓它實際跑 parent 的 Opus。

另在全域 AGENTS.md 加一條輕量 delegation 規則（探索、實作、測試、重複修改交給 subagent；架構決策與困難 debug 留在主 agent）。Opus 不一定會主動 delegate，沒有這條規則時 Sonnet worker 可能根本不出場。

## 2026-10-21 驗證

- CLI 2.1.302：`claude -p --model opus --output-format json`，要求 spawn `model: opus` 的 subagent；`modelUsage` 顯示 `claude-opus-5-5`（主）與 `claude-sonnet-5-5`（subagent）。
- Claude Desktop Code 分頁 2.1.304：subagent 指定 `model: opus`，回報自己是 `claude-sonnet-5-5`。

## Desktop 與 CLI 的關係

- Desktop Code 分頁跑的是 app 內建的 Claude Code（`~/Library/Application Support/Claude/claude-code/<version>/`），不經過 `~/.local/bin/claude` wrapper，版本跟著 app 更新，可能與 CLI 不同。
- 兩者共用 `~/.claude/`（settings.json、CLAUDE.md、agents、skills、plugins）。settings.json 的 `env` 在 Desktop session 程序環境中確實生效。
- Desktop 用啟動參數傳入 app UI 選的 `--model`、`--effort`（例：`--effort=medium`，環境變數 `CLAUDE_EFFORT=medium`），會覆蓋 settings.json 同名設定。要兩邊都生效的環境變數放 settings.json `env`。
- transfer_MAC 的 `sync-ai-agent-configs.py` 只合併 settings.json 的 `hooks`，不會清掉 `env`。

## Desktop `start_session` 不受此設定影響

Desktop 的 `start_session`（linked/side session）開的是獨立 session，不是 subagent，不讀 `CLAUDE_CODE_SUBAGENT_MODEL`；沒指定 model 時繼承 parent 的 model 與 effort（2026-10-21 實例：parent Opus/xhigh → 子 session `claude-opus-5-5`/xhigh）。

- 事後補救：`set_session_model` 只從下一個 turn 生效，當前 turn 仍跑 Opus。
- 現行做法：全域 AGENTS.md 一行規則 `New sessions (start_session) use Sonnet.`，屬指示而非硬鎖。

## 2026-10-24 改回：拿掉 FORCE

使用者要讓官方預設用 Haiku 的地方恢復原狀，Mac 的 `~/.claude/settings.json` 已移除 `CLAUDE_CODE_SUBAGENT_MODEL_FORCE`，保留 `CLAUDE_CODE_SUBAGENT_MODEL=sonnet`（備份 `settings.json.bak-20261024`）。fleet 其他主機未同步，也還沒在新 session 實測。

官方文件（sub-agents.md、model-config.md、env-vars.md，2026-10-24 查）：

- subagent model 解析順序：spawn 參數 → frontmatter `model`（`inherit` = 主 model）→ `CLAUDE_CODE_SUBAGENT_MODEL` → 主 model。FORCE=1 會忽略前兩者。
- 沒有 FORCE 時：`claude-code-guide` 依 frontmatter 跑 Haiku；general-purpose 走 env 的 Sonnet；Explore、Plan 跟主 model（Explore 已不再預設 Haiku，舊文件過時）。要 Explore 用 Haiku，得自訂同名 `Explore` agent 並寫 `model: haiku`。
- 背景任務（`--resume` 摘要、`/usage` 等）用 `ANTHROPIC_DEFAULT_HAIKU_MODEL`；`ANTHROPIC_SMALL_FAST_MODEL` 已 deprecated。Mac 未設這兩個，即預設 Haiku。
- Haiku session 進 plan mode 會升級成 Sonnet；`opusplan` 是 Opus 規劃、Sonnet 執行，沒有 Haiku 執行的混合模式。

## 未確認

- subagent 的 effort 預設值：Desktop binary 有 `subagentEffort` 字串，但未追到預設來源；不要假設一定是 medium。
- CLI 無 `effortLevel` 時的預設 effort：GrowthBook 快取同時有「Opus 建議 medium」與「high 為預設」兩種提示，未實測確認。
