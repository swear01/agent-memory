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

Bash 的 `PIPESTATUS` 必須在管線結束後立即保存在已準備好的變數。先跑 `local` 或 logging 都會改寫狀態。BenchExec 曾完成全部案例，卻因這個順序讓收尾報 `pipeline_status` unbound。這個案例只證明測量已完成，沒有修正後的報告生成證據。用前段失敗、尾段成功的 fixture 驗證。原始紀錄見 `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/shell/pipestatus-capture-before-command.md`。
