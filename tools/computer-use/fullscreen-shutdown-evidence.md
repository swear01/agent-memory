---
title: "黑畫面與關窗動作不能證明遊戲程序已退出"
scope: "tools/computer-use"
status: active
updated: 2026-09-07
evidence_digest: 1ba18f34dc71f28b29d0e7890aae0d254143c2ccdbd6882a2a80ca1a3ff828f6
---

# 黑畫面與關窗動作不能證明遊戲程序已退出

## Problem

歷史診斷回報黑色全螢幕後 Java 仍活動，render thread 等待 GLFW 事件、server thread 仍 tick。

## Durable rule

以目標程序狀態和當次 shutdown/world-save 記錄確認退出；不得只憑黑画面或關閉操作宣稱已停止。此來源沒有證明特定 F11 操作或外部 bug 就是修復方案。

歷史來源保留當時觀察，未重跑或驗證修復。
