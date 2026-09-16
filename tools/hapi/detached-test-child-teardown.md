---
title: "測試隔離也必須回收 detached child"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 301b715a067d3ed9596488249732dc916123756b152fffaa888068f494ad4c0d
---

# 測試隔離也必須回收 detached child

歷史調查回報一般 suite 重新納入啟動真實 detached session 的 integration test，暫存 Hub／home 隔離了正式資料，卻未統一回收 test child，刪 home 後留下 orphan。

生產 session 保活與測試 teardown 是不同責任；保存 test-owned PID／process group、分離 worker state、限制繼承環境，確認退出才刪暫存目錄。來源只回報調查及清理，未改 runner／測試；用戶原先要求先調查，清理回報也不能回填為事前授權或永久修復。
