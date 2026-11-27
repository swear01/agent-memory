---
title: CLIProxyAPI fleet 維運：Swear Review、session 與實際部署證據
scope: projects/deepseek-latch-gateway
status: active
updated: 2026-11-12
---

- **換 Gateway 連 Swear Review 都停掉。** 使用者只退役自製 DeepSeek Latch Gateway；Review、專用 tunnel、GitHub App 與最新 head gate 保持。正式走 transfer_MAC 的 CLIProxyAPI renderer；舊 Bun router 的 config／/status 不套用。先用真正 open PR full review 驗 caller 跑通新 gateway，再關已授權退役的舊元件。Health、gate-off success 或已合併 PR 留言都不能證實 inference 或 review 無 findings。
- **Session 已有 x-opencode-session，路由仍隨 prompt 變。** 同時保留上游 `x-opencode-session` 與 CLIProxyAPI 的 `X-Session-ID`／`X-Session-Affinity`。Pi 的 `compat.sendSessionAffinityHeaders=true`、`sessionAffinityFormat=openrouter` 供這兩個模型使用。用不同 session 發完全相同正文，再驗同 session 追加訊息及 compaction，避免把內容 hash fallback 當 session affinity。官方 API 402 與同機 local gateway 是不同路徑。
- **改 slot 或 key 後模型仍走舊設定。** 查 live instance 的 port、binary、private keybag 和 watcher；Zeus 兩帳號各自確認 35001／35002。Command Code 第三槽用 Unix `COMMAND_CODE_API_KEY_3`，Windows 是既有本機 DPAPI／SYSTEM launcher。採 canonical template，勿在 Windows 另維一套變數。先限制父目錄與繼承 ACL 再寫私人設定，Linux `<home>/.secrets` 用 0600 bare assignments 並保護 allexport，renderer 靜態讀取不 source/eval。官方 runtime／架構、loopback、401 未認證和非空 Flash/Pro 分開驗；別因 NFS 共用檔案變更而宣稱各程序已載入。
- **Mac 換 binary 後 checksum 相同卻 SIGKILL。** 從已核對 archive 建新 inode再原子替換，保留舊檔。每台只 drain/restart 正確的 gateway；不是把 HAPI 和子 session 都重啟。改 secrets 檔不會移除現行 runner env；若工作確實要改 Runner 再依該機 supervisor 保留子 session 手法處理。
- **模型清單有 Flash、實際回空或換上游。** 用帶 session 的真實 completion 確認非空內容、served model、gateway active endpoint/attempt 和 usage。失敗保留同欄位，診斷 1010、欠款、配額、憑證撤銷與 client 取消，勿以重啟假裝清額度。[NVIDIA／Radeon 與 renderer 陷阱](radeon-fleet-fallback.md)

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/deepseek-latch-gateway/cliproxyapi-retirement.md) · [舊 Latch Gateway 路由、failback 和舊 config](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/deepseek-latch-gateway/fleet-routing-and-failback.md)
