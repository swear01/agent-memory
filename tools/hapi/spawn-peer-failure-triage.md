---
title: spawn-peer 失敗判讀：502 是 worktree 建不起來，500 是停不掉的殭屍 session
scope: tools/hapi
status: active
updated: 2026-11-11
tags: [hapi, spawn-peer, stop-peer, worktree, runner-log, zombie-session]
---

# 症狀與判讀（2026-11-11 在 Mac runner 0.30.7.7 實測）

使用者說「`hapi spawn-peer` 壞了」時，先分辨三種完全不同的東西，不要直接重啟：

1. **spawn-peer 本身通常沒壞。**逐一測過 `--agent claude`、`--agent agy`、`--machine <linux 機>`、
   `--permission-mode bypassPermissions`／`acceptEdits`、`--session-type worktree`（在真 git repo 內），
   全部 `ok:true`，peer 也真的回了訊息。要宣稱壞掉前先跑這幾個對照。
2. **`spawn_failed: HTTP 502`＝child 建立階段被 runner 拒絕，hub 把原因吞掉了。**
   本次真因是 `--session-type worktree --dir <非 git 目錄>`：runner log 有
   `Worktree creation failed: Path is not a Git repository: fatal: not a git repository`，
   CLI 只印 `HTTP 502`。所以 worktree 模式的 `--dir` 必須是 repo 本身（見
   `spawn-peer-worktree-grouping.md`），指到 `~/Documents` 這種父層就長這樣。
3. **`stop failed: HTTP 500`＝該 session 的退出無法被確認，變成殭屍。**

# 殭屍 session 的實際樣貌

- `hapi inspect-peer <id>`：`active: false`、`lifecycle: (none)`、`messages: 0`，看起來「已經停了」。
- `hapi stop-peer <id>`：HTTP 500；`delete-peer` 說 `Cannot delete active session. Archive it first.`
- Runner log 從建立當天起**每 5 秒**重試一次，永遠同一句
  `[RUNNER RUN] Session <id> not found without verified exit`。本次該 session 建於 10/29，
  到 11/11 已 94,082 次，最近 4000 行 log 裡 829 行都是這個迴圈。
- 後果：runner 一直被這個迴圈佔用、log 檔數與容量暴增、所有 hub 操作都變慢，
  但畫面與 `inspect-peer` 都不會告訴你原因。這與
  `session-lifecycle-exit-confirmation.md` 的 exit-confirmation 規則同源：
  「沒有 verified exit 就不算停掉」。

清僵屍只有重啟 runner 有效（會斷掉該機所有 session，含自己），所以先問使用者，
不要為了清一個 session 直接重啟。

# 排錯順序（照這個順序，省時間）

1. `hapi spawn-peer ... --json`：失敗時只有 `error.message`，資訊不足，別停在這裡。
2. 記下失敗時間，直接讀 runner log（檔名 `~/.hapi/logs/<date>-pid-<pid>-runner.log`，
   取 mtime 最新的那一個），用時間戳對齊：`grep "Worktree creation failed\|Spawning session"`。
3. `hapi inspect-peer <id>` 看 session 是否真的活著；`active:false` + `lifecycle:(none)` +
   `stop-peer` 500 就是殭屍。
4. 檢查 log 目錄規模：**不要用 `ls`**（本次 168 萬個檔案，`/bin/ls: Argument list too long`）。
   用 `python3 -c 'os.scandir'` 數，或用 `du -sh ~/.hapi/logs`。本次是 7.4 GB。

# 自己測試留下的 session 一定要收乾淨

`spawn-peer` 建的 probe session 是 active 的，`delete-peer` 會拒絕；順序固定是
`archive-peer <id>` → `delete-peer <id>`。worktree 模式還會留下 `<repo>-worktrees/<MMDD-xxxx>`
目錄與 `hapi-<MMDD-xxxx>` branch，要 `git worktree remove --force` 收掉。
