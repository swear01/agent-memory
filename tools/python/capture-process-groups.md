---
title: 隔離 subprocess 的 capture 必須處理父程序中斷
scope: tools/python
tool: Python subprocess
status: verified
updated: 2026-09-08
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


## Nested capture 的修復與驗證（issue212 / PR213）

單層 TERM/INT regression 通過，不能直接推廣到 outer capture → launcher →
inner capture → separate-session leaf。2026-09-07 的 harmless nested probe
確認：leader 收到 TERM 先退出時，outer 的 `proc.wait()` 立即回傳，接著
`killpg(SIGKILL)` 可殺掉尚未轉送訊號或寫 sidecar 的 inner capture。其另開 session
的 leaf 因而逃出原 group；實測 inner sidecar 缺失且 leaf 仍在 S 狀態，最後由
coordinator 核實 PID/command 後單獨清理。

因此不能從 outer 已退出／outer sidecar 已寫出，宣稱內層工作全部停止或紀錄完整。
Nested supervision 必須另外驗證 leader-first exit、內層的 TERM/grace/flush 及 leaf
消失，並避免外層 escalation 先殺掉負責清理的內層 supervisor。PR213 已修復並通過實際 nested regression，merge
`2c6689c932bf6259d646bb1e525db34c9d16e21e`。TERM 後保留完整 grace deadline，
不因 leader 提早退出而立即 KILL，也不在 grace 內先 reap leader 後再用其 PID 發訊號。
CLI `--termination-grace` 預設10秒，外層明確15秒讓內層10秒完成 cleanup/record flush。
CLI wall limit 必須正且有限，grace 非負且有限；NaN/Infinity 不可通過 float validation。

Root 獨立 outer SIGINT probe 使用內層預設10秒、外層15秒、延遲2秒後正常退出的
leaf；外/內 interrupted sidecars 均存在，leaf exit0 且程序消失，outer 傳回原SIGINT，
未需額外清理。production source SHA
`2de0011b09de9db759593b0755091ac64aae397b08c0e365a697f68ec47cf06b`。
外層 group 消失不足以推論其他 session 的 descendants 完成，所以不能只為縮短等待
加入早退 polling。這不涵蓋不可捕捉的 SIGKILL 或無法立即結束的 kernel D-state I/O。

測試 fixture 也要負責 setup failure：launcher 使用阻塞的 `subprocess.run(inner)`
即可，不要在它回傳後永遠等待未必會建立的 leaf marker。finalizer 先 TERM、bounded
wait，再處理必要的 KILL fallback；只殺 outer wrapper 會讓獨立 group 留下孤兒。
六個 CLI 非有限值案例用 `--name=value` 單獨改一個參數，避免另一個 invalid 參數讓
測試因錯誤原因通過。active notes 的 cleanup/API 說明與實作一起更新。

最終 head 的 Codex review 明確無重大問題；早期118個 core-only tests 與後來新增
setup-failure 等8個 focused checks 是分開的執行，不虛構成一次119-case full run。
