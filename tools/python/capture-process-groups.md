---
title: 隔離 subprocess 的 capture 必須處理父程序中斷
scope: tools/python
tool: Python subprocess
status: verified
updated: 2026-09-06
---

`Popen(start_new_session=True)` 讓 verifier 不會跟著 capture 收到終端訊號。只在
`TimeoutExpired` 分支清理，SIGINT/KeyboardInterrupt 或預設 SIGTERM 都能留下孤兒
verifier；父 shell 釋放 lock 不代表 verifier 已結束。

在 main thread 暫時接管 SIGINT/SIGTERM，spawn 前收到的訊號先記錄，等 Popen 回傳並
取得 process handle 後才中斷。中斷與 wall timeout 共用 process-group TERM→有界 grace
→KILL→wait 清理；清理中的重複訊號只記錄，不再丟 exception。leader 先退出也不代表
整個 group 已停止。清理完成再恢復 handlers 並傳回原訊號。

raw verifier log 保持原樣；被中斷的 capture 不應產生代表正常完成的 execution sidecar。
保留 partial evidence，後續 gate 必須拒絕將它當成功或合成 UNKNOWN。SIGKILL 無法由
Python handler 捕捉，不能宣稱這個方法涵蓋它。

驗證來源：CPAchecker PR184 review5122874730、PR196 commit
`cd7dd732f294e8322b7217153048e6b456aa007e`。真實 SIGINT/SIGTERM regression 使用忽略
TERM 的 child，並在 cleanup 期間重送訊號：原版兩案均留下 child；修正後84個相關
harness tests通過，確認 child 已被 reap、log 未改、completion sidecar 不存在。

後續整合：PR184 `f6baafe6d7e44daa0ad16ae9d51dc019e834f61a` 在 cleanup 後、
handlers 還有效時寫入明確 `termination_reason=interrupted` sidecar，保存實際 child
exit/signal、elapsed 與 log hash，再傳遞原訊號；classifier 一律 infrastructure_error，
即使 raw log 已有 TRUE。此契約取代上述初版「無 sidecar」行為；缺少或損壞 sidecar
仍不可接受。PR196 `58ec9c2a11d616b9d4a7bef6a55263ae2aeb265f` 保留 coordinator
程式 bytes 並補 recovery 文件，整合111測試通過（含 leader 提早退出的 descendant）。
