---
title: Claude Desktop 最近專案清單把 Google Drive 路徑的 @ 顯示成全形 ＠
scope: tools/claude-desktop
status: verified
created: 2026-11-03
updated: 2026-11-03
tags:
  - change_directory
  - google-drive
---

# 現象

- Claude Desktop 的 Code 分頁在 system prompt 列出的 recent project folders 中，Google Drive 路徑寫成 `<home>/Library/CloudStorage/GoogleDrive-<account>＠gmail.com/...`，`@` 變成全形 `＠`（U+FF20）。
- 直接拿這個字串呼叫 `change_directory`，會得到 `PreToolUse:mcp__ccd_directory__change_directory hook error: The requested directory could not be resolved.`

# 解法

- 先 `ls <home>/Library/CloudStorage/` 確認實際資料夾名稱（磁碟上是 ASCII `@`），把 `＠` 換成 `@` 再呼叫 `change_directory`，就能成功切換。
- 這個錯誤不代表資料夾不存在，也不需要使用者手動選資料夾。
