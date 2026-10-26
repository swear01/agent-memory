---
title: oracle 磁碟清理：哪些可以刪、哪些看起來像暫存但不能刪
scope: machines/oracle
tags: [disk, cleanup, qmd, swear-review, opencodereview, worktree, journald]
status: active
created: 2026-10-26
updated: 2026-10-26
---

2026-10-26 在 oracle（hostname `swever`，`/` 193G）把用量從 93G 清到 80G。之後再清可以照這份順序。

## 可以直接清

- `<remote-home>/.opencodereview/sessions/tmp-swear-review-job-*-repo`：swear-review 每次 bot review 留下的 repo clone，用完不會自動刪。這次累積了 1147 個、8.8G，是最大的一筆浪費。root cause 是 `runOcr` 先設 `HOME`、後展開 `ocrEnv`，結果真正的 HOME 蓋掉了 isolated HOME。已在 swear01/swear-review PR #15（`7a6d7ff`）修正，`/opt/swear-review/app` 要重新部署才會生效。刪之前先確認沒有正在跑的 review（`pgrep -af swear-review`，並且最近一小時沒有新目錄），然後用 `find ... -mmin +60 -exec rm -rf {} +` 清掉。
- journald：直接 `journalctl --vacuum-size=200M` 會釋放 0B，因為空間都在 active journal 檔。要先 `sudo journalctl --rotate` 再 vacuum，1.3G 才會降到 178M。
- `<worktree-root>/hapi/*`：每個 worktree 的 `du` 約 1.7–2.2G，但 bun 的 hardlink 讓單獨刪一個 worktree 實際釋放得比較少（8 個約 5G）。只刪同時符合「`git status --porcelain` 為空」以及「`git branch -r --contains HEAD` 有結果」的 worktree，並從 `<home>/hapi` 執行 `git worktree remove`。注意 repo 是 `swear01/hapi`，查 PR 時不要用上游 repo 名。

## 看起來像暫存，但不能刪

- `/var/tmp/qmd-memory-swever/`：這是 shared-memory 現役的 QMD config 和 index（`QMD_CONFIG_DIR`、`XDG_CACHE_HOME` 都指向這裡）。`cache/qmd/models` 裡的三個 gguf（embedding、reranker、query expansion，共 2.4G）都在用，`cache/qmd/index.sqlite` 和 `config/qmd` 也是現役的。其他像 `cache-*`、`config/<test-name>`、`index.sqlite.*.bak`、`*.snapshot.sqlite-*` 都是測試留下的副本，2026-10-26 已清掉，整個目錄剩 2.5G。

## 已刪除的 issue #25 資料

- 使用者確認後，2026-10-26 刪掉了 `/var/tmp/shared-memory-issue25-20260821-oracle/`（2.9G，redacted corpus 和 episodes）和 `/var/tmp/candidate-v3/`（0.9G shards）。當時 agent-memory issue #25 雖然還是 open，但從 2026-08-21 之後就沒有進度。要恢復的話，從原始 session log 用 `redact_jsonl.py` 重新產生。

## 殘留程序

- 2026-08-14 有一次 dsh 測試用 `nohup node_modules/.bin/dsh --profile web --patch hapi-no-web.yml --port 0 &` 在 `/tmp/dsh-runtime-test` 啟動，之後沒有 kill。它在 pm2 cgroup 底下掛了兩個多月，監聽 127.0.0.1 的隨機 port。agent 測試背景 server 時，結束前要記得 kill。
- agent 跑 Android build 會留下 Gradle daemon（約 0.5G RSS），閒置 3 小時會自己結束，不需要處理。
