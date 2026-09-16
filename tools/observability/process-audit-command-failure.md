---
title: "程序稽核命令失敗不能當成零洩漏"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: 8adb3104d5dba1cf392c54902dd6687a1a037a2377f0276d374b95536f821e39
---

# 程序稽核命令失敗不能當成零洩漏

歷史助手找不到預期 marker process，追查才發現 ps eww -axo 在該 Linux procps-ng 版本報錯，先前稽核因此沒有有效程序清單。

先檢查列舉命令的退出碼、stderr 與基本可觀測性，再解讀零匹配。使用目標平台確實支援的選項，不把空輸出當成乾淨證據，也避免把環境憑證輸出到日誌。來源只支持當時改查可用選項，未證明後續稽核通過。
