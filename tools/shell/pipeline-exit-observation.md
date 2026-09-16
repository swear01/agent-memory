---
title: "管線末端成功不能代表被測程式成功"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: 636c80a8b9b47cc4c3335e212d58b9ad33abd95eef597f760319ad41f7fb2219
---

# 管線末端成功不能代表被測程式成功

Strict 測試已出現 401 traceback，畫面卻顯示 rc=0；助手指出該值來自末端 tail。後續來源明確顯示 Python rc=1，而且 strict 輸出檔不存在。

驗證 CLI 的失敗契約時，記錄目標程序本身的退出狀態與產物狀態。若保留管線，使用該 shell 正確的各段狀態或失敗傳遞機制；不要把最後一段成功當成前段成功。這段證據只涵蓋 strict 路徑，不代表其他降級模式應採相同退出碼。
