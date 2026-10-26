---
title: 關閉 Unity Editor 後殘留的子程序（Dock 殘影、授權掉線）
scope: tools/unity; macOS; 多個 agent 驅動的 Unity 6000.6 Editor
status: active
created: 2026-10-26
updated: 2026-10-26
---

- 症狀：Editor 已被 `kill -TERM` 結束，Dock 仍顯示「Unity 正在背景中執行」。`lsappinfo info <ASN>` 顯示該 Editor pid 為 `(exited-with-subordinates)`。
- 根因：WebGL build 會啟動 `PlaybackEngines/WebGLSupport/BuildTools/Emscripten/node/node .../websockify/websockify.js`。Editor 死後它被 launchd 收養（ppid 1）繼續執行，讓 LaunchServices 保留該 Editor 的 app 紀錄與 Dock 圖示。殺掉這個 websockify node 後，圖示立即消失。
- `AssetImportWorker` 子程序也可能在 Editor 結束後殘留，要一併處理。
- 清理順序：
  1. 先用 `ps -axo pid,ppid,command | grep -i LicensingClient` 確認要關的 Editor 不是共用授權 client 的父程序，否則其他 Editor 會一起掉授權。解法見 [[licensing-client-launchagent]]。
  2. 再 `kill -TERM` 該 Editor（`-projectPath` 對應 worktree）。
  3. 最後收掉它的 workers 與 `Editor/6000.*/.../websockify.js` 孤兒程序。
- 正在被使用的 GUI Editor 出現在 Dock 是 Unity 的正常行為。只有 Hub 版的 Licensing Client 設了 LSUIElement/LSBackgroundOnly。
