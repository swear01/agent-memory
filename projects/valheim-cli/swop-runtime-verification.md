---
title: ValheimCli SWOP runtime and release verification
scope: projects/valheim-cli
status: verified-with-limits
updated: 2026-10-12
---

- **Bridge 已安裝但 CLI 或 skill 指令不符。** C# Bridge／Thunderstore 和原生 Rust CLI 分開版本。0.5.0 Bridge 同一 loopback 28761 提供帶本機 token 的 HTTP `POST /v1/command` 及 framed TCP，兩邊共享權限和 UUID 重複寫入防護，HTTP 無 CORS。原生 CLI／HTTP 不需要 Node，只有可選的 npx 安裝器需要；skills.sh 個別頁以 SKILL.md 為來源。ValheimCliBridge 舊包改名需先卸避免雙 DLL，plugin GUID 保留。
- **世界載入時收到 `Expired before execution; action cancelled`。** 等 ready 後可重試唯讀。寫入執行狀態不明時檢查原遊戲狀態與同 UUID，勿生成新 UUID 盲重送。若外層 harness deadline 短於 launcher，延長外層再排查。AllowTeleport 可在 Configuration Manager 即時切換，Bridge Enabled／Port 需重啟；token 留 profile。
- **背景操作看似通過，就宣稱所有平台或多人可用。** 用 SWOP 隔離 Gale profile、獨立角色與世界驗原 GUI IDs、遮擋／過期拒絕、lease 停止、暫停後拒輸入、緊急停、保存登出與 listener 結束。校準觀察器另驗前景、游標和 console，兩種傳輸交叉核重複寫入。2026-10-12 已驗上述隔離 Windows 路徑，多人、dedicated server、Linux/macOS 實機、所有原生／第三方動作與長時間仍未涵蓋。
- **社群首頁有舊 cache，或 build 只有 prerelease。** 用 [Thunderstore ValheimCli](https://thunderstore.io/c/valheim/p/swear01/ValheimCli/) 版本頁及 API 讀回，再取公開 ZIP 比 DLL hash。GitHub Release/Latest 另讀回。社群 ZIP 不含 token、設定、日誌或遊戲 DLL；公開附件和現行實測範圍各自核對。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/valheim-cli/swop-runtime-verification.md)
