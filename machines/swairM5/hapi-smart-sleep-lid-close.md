---
title: SwairM5 合蓋防睡眠：HAPI smart-sleep 與 Claude 活動偵測
scope: machines/swairM5
machine: SwairM5
tags: [pmset, lid-close, clamshell, hapi, claude-code, launchd]
status: active
created: 2026-10-25
updated: 2026-10-25
---

# 結論

- 真正能擋住「合蓋睡眠」的只有 `pmset disablesleep 1`（`pmset -g` 顯示
  `SleepDisabled 1`）。Claude 桌面版的 keep-awake、`caffeinate`、Amphetamine 類
  assertion 只擋 idle sleep；電池合蓋時 `pmset -g log` 仍會出現 `Clamshell Sleep`。
- SwairM5 已有自製的 HAPI smart-sleep，只在「有工作」時切 `disablesleep`，
  2026-10-25 起也把 Claude Code 的活動算進去。

# 架構（`~/Library/Application Support/HAPI/`）

- 使用者 LaunchAgent `xyz.hapi.smart-sleep-watcher`：`/usr/bin/python3
  hapi_smart_sleep.py`，每 2 秒把 `1`/`0` 寫到 `smart-sleep.state`。
  `ActivityLatch`：busy 即開；idle 10 秒或 unknown 60 秒後關。
- root LaunchDaemon `xyz.hapi.smart-sleep-helper`：
  `/Library/Application Support/HAPI/hapi-smart-sleep-helper.sh` 讀 state
  （超過 20 秒視為過期），切 `pmset disablesleep`，結束時重設為 0，
  log 在 `/var/log/hapi-smart-sleep.log`。
- `xyz.hapi.auto-virtual-display`（`lid-display-switch`，BetterDisplay 虛擬螢幕）
  只負責合蓋／開蓋切螢幕，不防睡眠。

# Claude 活動偵測

- Claude 桌面版的 `claude` 行程會常駐好幾天，不能用「行程存在」判斷。
  改用 `~/.claude/projects/*/*.jsonl` 的 mtime：CLI、桌面版 Code 分頁
  （含 Remote Control）、HAPI 的 claude session 都會寫這裡。
- `claude_recently_active()`：任一 transcript 在 `CLAUDE_ACTIVE_SECONDS = 180`
  秒內有寫入就算 busy，放在 `observe_activity()` 最前面；log 會顯示
  `HAPI/Claude activity busy; closed-display sleep blocked`。
- 沒涵蓋：桌面版一般聊天（跑在雲端，不需要）、Cowork（本機沒有 jsonl）。
- 已知限制：單一工具呼叫靜默超過 180 秒會讓 Mac 中途睡著；
  電池合蓋放包包時只要 Claude 在跑就不會睡（發熱、耗電）。

# 修改與驗證

- 修改後重啟：`launchctl kickstart -k gui/$(id -u)/xyz.hapi.smart-sleep-watcher`。
- 測試：同目錄 `python3 -m unittest test_hapi_smart_sleep`（11 個通過）。
- 已實測 busy 時 `SleepDisabled 1`；切回 0 只有單元測試覆蓋。
- 原檔備份：`backups/*.20261025`。
