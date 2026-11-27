---
title: Mac 還原區分可搬資料、認證與實際可用
scope: global/migrations
status: active
created: 2026-08-18
updated: 2026-11-28
tags: [migrations, macos, secrets, restore, verification]
---

- **整份 profile 或備份看起來可移植。** 只恢復核准的 scripts、templates 和非機密偏好；不搬 SSH 私鑰、OAuth/browser cache、Keychain 或登入狀態。曾進入明文備份的秘密依曝露處理，輪替或撤銷；`0600` 只用來保護新值。
- **重建覆蓋了目的端的設定或 symlink。** 逐個 preflight，保留目的端原來的檔案、目錄及 symlink，再複製允許清單。讀回偏好檔只驗該項恢復。
- **產物已生成，卻尚未安裝或啟動。** 分開回報 `artifact_ready`、`syntax_checked`、`runtime_verified`。保留安裝器 nonzero，實際操作安裝後的應用程式才算可用。Homebrew 一次只跑一波，結束再並行獨立的非 Homebrew 任務。

[原始遷移試作與證據界線](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/global/migrations/portable-restore-boundary.md)
