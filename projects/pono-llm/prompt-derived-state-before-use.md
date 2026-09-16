---
title: "Prompt 的衍生狀態要在所有使用路徑前定義"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 95cfc63e2118daab9b866a7f7479fc293bccfb5fe9e9df937687a33b2ae589fd
---

# Prompt 的衍生狀態要在所有使用路徑前定義

使用者 traceback 直接顯示 build_software_prompt 在 if has_arrays 讀到未賦值的區域變數；相鄰來源另有文字替換找不到目標的錯誤。

從實際解析出的 state 結構先計算 prompt 所需條件，再走 array／scalar 分支，以觸發原錯誤的輸入驗證。來源片段沒有完整賦值位置或修復後結果；不能把替換失敗與 UnboundLocalError 合成同一已確認根因，也不應把寬度零直接當普通 scalar。
