---
title: 隔離 subprocess 的 capture 必須處理父程序中斷
scope: tools/python
tool: Python subprocess
status: verified
updated: 2026-09-08
---

## 單層 capture

`Popen(start_new_session=True)` 讓子程序群組不會收到父終端的訊號。只有 `TimeoutExpired` 會清理的 harness，遭 SIGINT/SIGTERM 中斷會留下孤兒；父 shell 釋放 lock 不足以證明 verifier 已退出。

- Main thread 暫時管理 SIGINT/SIGTERM。Spawn 尚未取得 handle 時先記訊號，取得後再導向清理。中斷和 timeout 用同一條 TERM → 有界 grace → KILL → wait，清理中的重複訊號只記錄，完成後恢復 handler 並傳遞原訊號。
- 原樣保留 raw verifier log。後續契約應記 `termination_reason=interrupted`、child exit／signal、elapsed 與 log hash，分類為 infrastructure_error，即使 raw log 已出現 TRUE。早期版本無 sidecar；缺失或損壞都不可當正常完成或自造 UNKNOWN。
- 實際用忽略 TERM 的 child 測 SIGINT/SIGTERM、清理期間重送、中斷 sidecar 和 log 保留。SIGKILL 不可捕捉，D-state I/O 也未必即時退出。

## 外層與內層各有群組

歷史復現中 outer leader 收到 TERM 後先退出，`proc.wait()` 返回便立即 KILL，反而殺掉還在轉送和寫 sidecar 的 inner capture；另開 session 的 leaf 仍存活。外層保留完整 grace，不因 leader 退出提前 escalation 或先 reap 再重用 PID 發訊號。給內層完成 TERM/grace/flush 的時間。已驗證案例內層預設 10 秒、外層明確 `--termination-grace 15`；按實際層次配置。

排程測試須驗 leader-first exit、inner/outer interrupted sidecars、獨立 leaf 確實退出、outer 傳原信號。setup failure fixture 可由 launcher 用阻塞 `subprocess.run(inner)`，不要完成後還等未必出現的 leaf marker。Limit 必須正且有限，grace 非負且有限。NaN/Inf fixture 一次用 `--name=value` 改一個欄位，以免因別人的非法值提早拒絕而假綠。

[完整原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/python/capture-process-groups.md)
