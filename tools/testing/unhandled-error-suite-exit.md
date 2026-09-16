---
title: "Assertions 全過仍須檢查測試程序退出碼"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 8b21a58bd93c101368de5d390b49909bafd576c6fbdd7f56385615a89d3f1dbf
---

# Assertions 全過仍須檢查測試程序退出碼

歷史助手回報 CI 的 assertions 全過，Vitest 卻因兩個 window is not defined 的 unhandled async errors 退出1；本地同分支則沒有錯誤，重跑要求又被拒絕。

同時讀取 assertion、未處理非同步錯誤與退出狀態，避免只計算 passed tests。未修改錯誤所在檔案或本地通過，不足以證明 CI 失敗與變更無關；來源沒有上游對照或成功重跑，既有 flake 仍是猜測。
