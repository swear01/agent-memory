---
title: "Oracle 磁碟壓力：QMD 保護、worktree 和 journal"
scope: machines/oracle
tags: [disk, cleanup, qmd, swear-review, opencodereview, worktree, journald]
status: active
created: 2026-10-26
updated: 2026-11-28
---

# Oracle 磁碟壓力：先保留現役環境，再定位來源

- `/var/tmp/qmd-memory-swever/` 同時有 QMD 應用設定、索引和當前模型，即使位於暫存根目錄也要先核對 live `QMD_CONFIG_DIR`、`XDG_CACHE_HOME` 和 process。與試驗用 snapshot／cache 副本、活 SQLite sidecars 分開盤點。對 Oracle 的查詢與向量重建用 [Oracle QMD 筆記](qmd-vsearch-slow.md)。
- 找到成批 `tmp-swear-review-job-*-repo` 時，先查 Swear Review 現役工作。舊 runner 的環境合併順序曾讓 `ocrEnv` 蓋掉隔離的 `HOME`，導致 repo clone 寫到共用 sessions。修正後以新 job 完成再查 sessions 是否累積。舊目錄即使長得像同一 bug，也逐批核對 owner、程序和已驗證來源，不能憑名字當刪除授權。
- Worktree 以 Git 本身檢查和移除。須驗無 active session／child、工作樹乾淨、所有 commit 有遠端對應分支或其他可恢復保存，而且沒有使用者保留標記。用 `git worktree remove`，不 rm repo 路徑。Bun hardlink 會讓單目錄 `du` 高估釋放量，使用 `df` 的 filesystem 讀回。
- Journald vacuum 只清 archive。大量空間在 active journal 時，核對保留要求後先 rotate 再 vacuum；以 `journalctl --disk-usage`、`df` 驗。追蹤自己啟動的暫存 server/process；確認原工作的 PID／session 已停，不因長期留在 pm2 cgroup 直接殺現役程序。

[Oracle 2026-10 專案收尾、容量和逐項佐證](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/oracle/disk-cleanup.md)

## Swear Review checkout 配額與工作目錄

- Git checkout 遇到 `Disk quota exceeded` 時，先用 `findmnt -T <workspace>` 確認掛載點，再讀服務帳號的使用者配額。Oracle 的 `/tmp` 是帶 `usrquota` 的 tmpfs，`df` 只顯示整個掛載的容量。2026-11-28 用 `quotactl_fd(Q_GETQUOTA)` 查到 `ubuntu` 的上限約 4.66 GiB。checkout 成功後才觸發的 45 分鐘 OCR 硬逾時應另查。
- 這次的 QMD 暫存副本約占 2.8 GiB。核對索引雜湊與開啟檔案後，搬到根磁碟上的 `/var/tmp`；跨檔案系統先複製、逐檔驗證，再移除原件並重新讀配額。正式 QMD 索引以原設定查詢成功。
- Oracle 正式設定 `/opt/swear-review/data/config.yaml` 的 `workers.workspace_dir` 已改為根磁碟上的 `/opt/swear-review/workspaces`（`ubuntu:ubuntu`、`0700`）。改路徑時用部署版設定載入器驗證，等 DB 無 queued/running/cancelling job，再備份設定並重啟 `swear-review`。回滾使用 `config.yaml.bak-workspace-*` 並重啟。這次驗了 healthz、readyz、實際 checkout 所在的 ext4、OCR 完成、GitHub 回報和工作目錄清理。

[2026-11-28 Swear Review 真實測試](https://github.com/swear01/swear-review-e2e/runs/140789223229)
