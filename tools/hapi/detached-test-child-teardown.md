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

## Runner 由自己的 shutdown 路徑回收

Session／terminal child registry 不宜殺 runner 本身；曾有 stage-2 tree-kill 先殺 runner，導致 `stopRunner` 的 graceful cleanup 被跳過，下一測讀到 stale PID。分開追蹤兩種程序的所有權，重用狀態時核對 PID 身分及實際可用性。若採 `pkill` 或字串比對，先排除清理 shell 自己，避免命令提前退出又留下程序。應逐項按本次擁有的身分回收、驗退出；兩個歷史證據未含最終修復。見 `git show 9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e:tools/hapi/test-cleanup-process-ownership.md`、`git show 9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e:tools/hapi/cleanup-self-matching-pattern.md`。
