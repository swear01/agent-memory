---
title: "鍵盤與截圖成功不代表滑鼠互動已驗證"
scope: "tools/computer-use"
status: active
updated: 2026-09-07
evidence_digest: 92b50304489bf2f67460ff07c97bdd95c9cece1c16e11a4e01ce0b214aebfd96
---

# 鍵盤與截圖成功不代表滑鼠互動已驗證

## Problem

Agent 回報 macOS 合成 mouse-down/up 被 LWJGL 忽略，native pipe 回報 sender unauthenticated；GUI shift-click 因此未完成。

## Durable rule

依互動類型分別記錄驗證結果。保留失敗操作與未完成的目視風險，不能以其他輸入或 GameTest 成功代替 GUI click 驗證。

歷史來源保留當時觀察，未重跑或驗證修復。
