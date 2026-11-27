---
title: "Oracle 磁碟壓力：QMD 保護、worktree 和 journal"
scope: machines/oracle
tags: [disk, cleanup, qmd, swear-review, opencodereview, worktree, journald]
status: active
created: 2026-10-26
updated: 2026-10-26
---

# Oracle 磁碟壓力：先保留現役環境，再定位來源

- `/var/tmp/qmd-memory-swever/` 同時有 QMD 應用設定、索引和當前模型，即使位於暫存根目錄也要先核對 live `QMD_CONFIG_DIR`、`XDG_CACHE_HOME` 和 process。與試驗用 snapshot／cache 副本、活 SQLite sidecars 分開盤點。對 Oracle 的查詢與向量重建用 [Oracle QMD 筆記](qmd-vsearch-slow.md)。
- 找到成批 `tmp-swear-review-job-*-repo` 時，先查 Swear Review 現役工作。舊 runner 的環境合併順序曾讓 `ocrEnv` 蓋掉隔離的 `HOME`，導致 repo clone 寫到共用 sessions。修正後以新 job 完成再查 sessions 是否累積。舊目錄即使長得像同一 bug，也逐批核對 owner、程序和已驗證來源，不能憑名字當刪除授權。
- Worktree 以 Git 本身檢查和移除。須驗無 active session／child、工作樹乾淨、所有 commit 有遠端對應分支或其他可恢復保存，而且沒有使用者保留標記。用 `git worktree remove`，不 rm repo 路徑。Bun hardlink 會讓單目錄 `du` 高估釋放量，使用 `df` 的 filesystem 讀回。
- Journald vacuum 只清 archive。大量空間在 active journal 時，核對保留要求後先 rotate 再 vacuum；以 `journalctl --disk-usage`、`df` 驗。追蹤自己啟動的暫存 server/process；確認原工作的 PID／session 已停，不因長期留在 pm2 cgroup 直接殺現役程序。

[Oracle 2026-10 專案收尾、容量和逐項佐證](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/oracle/disk-cleanup.md)
