---
title: swairM5 DVLab 網站本機 checkout 與 preview 現況
scope: machines/swairM5
tool: local-preview
machine: swairM5
status: active
updated: 2026-10-28
---

- Machine name 可寫作 `swairM5` 或 `SwairM5`；以下使用 `<remote-home>` placeholder，不保存 raw user-home path。
- 2026-10-28 家目錄清理後，唯一保留的 checkout 是 old CRA repo：`<remote-home>/Documents/dvlab-ntu.github.io`（detached `6e33b72`，有未提交的 `backend/server/server.js` 修改與 `frontend/build/`）。
- 已移除：Astro PR101 checkout `<remote-home>/dvlab-ntu.github.io-pr101`（分支與 origin 同步，工作已在 PR）、launchd 服務 `dvlab-astro-pr101-v4`（原本在 `:4321` 跑 preview，plist 已從 `~/Library/LaunchAgents` 移除）、CRA static mirror 與 zip（`:4174`）、`dvlab-local-mongo`、`dvlab-cra-ref` 截圖。使用者明確表示 mongo/CRA preview 用品不需保留。
- 需要再 preview Astro 站時，到 `<remote-home>/Documents` 或 `<remote-home>/.agent-worktrees/` 下重新 clone，不要放回家目錄，也不要再建常駐 launchd 服務。
