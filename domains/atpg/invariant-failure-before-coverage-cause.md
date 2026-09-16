---
title: "ATPG coverage 異常先處理可見不變式失敗"
scope: domains/atpg
status: active
updated: 2026-09-16
evidence_digest: aeacc7a4ebeb8e21dd8bcc9d6526f56edae14fa8fdb752acc5806819e15dc947
---

# ATPG coverage 異常先處理可見不變式失敗

使用者輸出中 storeCurrentAtpgVal 反覆報 numAssignedValueChanged 不為零，但程式仍 exit 0 並列 coverage；助手把大量 AU 歸因於強制 backtrack，來源未完成該因果查核。

把內部不變式錯誤與統計結果一併保存，不能以成功退出掩蓋，也不能只用 AU 數斷言 testable 被誤分類。定位狀態更新與回溯契約並重現後再解釋 coverage；這份來源另有截斷的 loop 診斷，沒有修復後數據或完整根因證明。
