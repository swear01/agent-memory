---
title: "NFS 大型建置樹探索要限制範圍"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: 2b6dcff8faf925ae7e97048af23f8849d1f6fbfc954545414d8ad961e9121636
---

# NFS 大型建置樹探索要限制範圍

使用者指出多個 broad find build 查詢卡在 NFS D 狀態，要求停止擴大探索，改用明確路徑完成必要 contract 檢查，再執行原本下一步。

先用已知產物與目錄縮小查詢範圍；換成 rg 也要限制路徑，不能假設工具名稱本身就讓遞迴掃描有界。等待檔案系統中的程序未必能立即中斷，仍需核對實際退出狀態。來源只有指令與故障回報，沒有後續恢復結果。
