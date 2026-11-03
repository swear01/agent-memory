---
title: Swear Review DeepSeek 額度消耗來源與改為手動觸發
scope: projects/swear-review
status: deployed on Oracle 2026-10-22 (app 86e5f9f, review.auto false)
updated: 2026-11-04
---

- 2026-10-22 查 Oracle 正式 DB（14 天）：push（`synchronize`）觸發的 full re-review 約占 review 時間 55%；開始後才被新 push 取消的 job 69 個、487 分鐘（約 25%）白跑。gateway 72 小時約 1 萬次請求，幾乎全打在 OpenCode Go 帳號 1。
- 修法一：`swear01/swear-review` PR #14（merge `86e5f9f`）讓 synchronize 改審 `last_successful_review_sha..HEAD`；無前次成功、compare 非 `ahead`（force-push）、compare 失敗或 gate 非 off 時退回 full；同一 head 重複 synchronize 直接略過。已知限制：PR branch merge main 時增量範圍會含 main 的 commit。
- 修法二：使用者要求只在留言時審，Oracle `/opt/swear-review/data/config.yaml` 改 `review.auto: false`（備份 `config.yaml.bak-pre-auto-off-*`），重啟後 healthz/readyz OK。DB 備份在 `data/backup-pre-push-incremental-*`（ubuntu 無權寫 `/opt/swear-review/backups`）。
- skills 同步：`shared-skills` PR #51 把 github-pr-review-loop / personal-pr-workflow 改成 Swear Review manual-only（`/swear-review` 或 `/swear-review incremental`）。
- 查用量的方法：`sqlite3 -readonly data/swear-review.db` 對 `review_jobs` 依 trigger/status 加總 `finished_at-started_at`；`curl 127.0.0.1:35001/status` 看各帳號 requests。

## Check 重跑的 full range

- Incremental job 的 `base_sha` 是前次成功 review 的 head，不是 PR base；`check_run.rerequested` 不能只把 mode 改成 full 後沿用此值，否則 merge-base 仍是增量起點，會漏審較早改動並誤標 full。
- PR #16（merge `4b376ba`，2026-11-04）改為讀取 live PR base；PR 已關閉或 live head 與原 job head 不同就忽略重跑，避免舊 check 請求取消新 head 的 review。130 tests、typecheck、build 通過；此紀錄僅證明修正已合併，不代表正式環境已部署。上方 status 保留 2026-10-22 的歷史部署證據。
