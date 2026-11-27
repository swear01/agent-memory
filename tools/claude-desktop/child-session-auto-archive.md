---
title: Claude 桌面版只讓 child session 在 PR 合併後自動歸檔
scope: tools/claude-desktop
tool: Claude desktop app
machine: SwairM5
status: active
confidence: high
created: 2026-10-25
updated: 2026-10-31
tags:
  - claude-desktop
  - auto-archive
  - plugin
---

## 只給已綁定 PR 的 child 開啟監看

- Desktop 全域 `auto_archive_on_pr_close` 不能依 parent/child 加條件。只要 child 時維持全域關閉，查 `get_session("self")` 的 `parentSessionId` 和 `detached`；不用對主 session 設同一規則。Child 可能被 detach，不能永久快取 child=true，也不能把讀取失敗快取為「不是 child」。
- 只對已綁定且仍 open 的 PR，用 per-session `ccd_pr` 的 `set_monitor({auto_archive_on_close:true})`；用 `get_status` 讀回 `monitor.auto_archive_on_close`，最後驗實際 archived。App 監看的是 PR 合併或關閉，因此實際包含兩者。PR 關閉後再設會被拒絕。設定已持久化後無需讓 session 常駐等待事件。
- `gh pr create`／`git push` tool.call hook 可能早於 App 判讀輸出及建立 binding；當時改在回合結束查。若同一 session 之後綁新 PR，需分開讀回。
- 在 Mac 歷史版，auto 模式拒絕 plugin 自行呼叫，log 為 `classifier gave no verdict ... the request that produced this action did not ask for one`。加 `permissions.allow` 未解；不要重試已被拒的動作。部署過「由 agent 在明確需要時呼叫」的版本；真實 auto child PR 尚未有端到端驗證。查 `main*.log` 的 `permissionMode ... effective=`，以及 `Session <id> auto-archive on PR close set to true`，勿把 plugin tests 當成已歸檔。
- 來源在 transfer_MAC `stow/agents/.agents/claude-plugins/mods/child-auto-archive`。當時 `claude plugin validate` 要求接收 `$` 的 helper 是檔案頂層 function，放 `register` closure 會載入失敗；在 register 重設模組層狀態。Session dev-mod 僅熱載入當次，folder marketplace 和 user scope 是不同發佈入口。

[完整原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/claude-desktop/child-session-auto-archive.md)
