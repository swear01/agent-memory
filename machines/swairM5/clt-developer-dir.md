---
title: swairM5 遇到 Xcode 授權協議阻擋時以 DEVELOPER_DIR 走 CommandLineTools
scope: machines/swairM5
machine: swairM5
tool: git
status: active
confidence: high
created: 2026-09-25
updated: 2026-09-25
tags:
  - macos
  - git
  - xcode
  - command-line-tools
---

# 問題現象

當 macOS 上 Xcode.app 升級但尚未完成授權協議確認時，在非互動式環境（例如 Agent session、腳本）執行 `/usr/bin/git` 或呼叫 git 的 CLI（如 `gh`）會卡在或報錯：
`You have not agreed to the Xcode and Apple SDKs license. You must agree to the license below in order to use Xcode.`

此為 macOS dev tools shim 依據 `xcode-select -p`（指向 `/Applications/Xcode.app/Contents/Developer`）進行授權檢查所致。非互動環境無法按下 Enter 捲動協議，導致指令 hang 住或失敗。

# 繞過與解法

無須 `sudo` 或中斷去開互動式終端機，只要環境變數指定 Command Line Tools 路徑：

```bash
env DEVELOPER_DIR=/Library/Developer/CommandLineTools <command>
```

`/Library/Developer/CommandLineTools/usr/bin/git` 具備完整的 Git 功能且不會要求 Xcode.app 授權協議。此設定對單次命令、背景任務或直接傳入 `gh`、`git` 均有效。
