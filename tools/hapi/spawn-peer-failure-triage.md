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

清僵屍**不是**只靠重啟 runner：重啟後 runner 仍然找不到那筆的退出證據，`unknown` 會繼續。
真正有效的是補 verified-exit tombstone 再重啟（見下方「已驗證的修復程序」）。
順帶更正一個常見誤解：**重啟 runner 不會斷掉 session**。`hapi runner --help` 就寫
`Stop the runner (sessions stay alive)`，macOS launchd 是 `AbandonProcessGroup=true`、
Linux systemd 是 `KillMode=process`。2026-11-12 實測：本機跑著的那個 session 經歷兩次
`launchctl kickstart -k` 全程存活，對話與程序都沒斷。

# 已驗證的修復程序（2026-11-12，Mac，清掉兩筆僵屍）

1. 先把**所有**卡住的 id 找出來（不要只看一筆）：
   `grep -o "Session [0-9a-f-]\{36\} not found without verified exit" <runner log> | awk '{print $2}' | sort | uniq -c`
   本次是兩筆：`15c75729…`（cwd 是帳號 home，建於 10/29）與 `43aee5f1…`（cwd 是 `<home>/Documents`）。
2. 備份 `~/.hapi/runner.state.json.verified-exits.json`（本次 392 筆，兩個 id 都不在裡面），
   把缺的 id append 進去（純 JSON 字串陣列）。
3. 重啟 runner：macOS `launchctl kickstart -k gui/$(id -u)/com.swear.hapi-runner`。
4. 驗證：新 runner log 的 `not found without verified exit` 次數為 0（原本每 5 秒一次）。
   只補其中一筆就重啟時，另一筆照樣每 5 秒重試 —— 這是「重啟不等於修好」的直接證據。
5. `hapi archive-peer <id>`（改判 `already_gone` 後才會成功）→ `hapi delete-peer <id>`。

上游 issue [#2408](https://github.com/tiann/hapi/issues/2408) 已開；正確修法應該是 runner 啟動時
把「上一輪實例追蹤過、現在沒有存活程序、也沒有 tombstone」的 session 自動寫成 tombstone，
而不是要人手動改 JSON。

# 502 的來源還沒完全定案（2026-11-12 狀態）

worktree 指到非 git repo 這個**觸發原因**是確定的（runner log 有
`Worktree creation failed: Path is not a Git repository`）。但「為什麼 CLI 只看到空洞的
`HTTP 502`」還沒查完，已知線索：

- 重現穩定：同一個指令約 30 秒後回 `{"ok":false,…,"error":{"code":"spawn_failed","message":"HTTP 502"}}`。
- 這台 Linux 機上的 hub（`127.0.0.1:3000`，`bun run <home>/hapi/hub/src/index.ts`）
  對 `POST /api/machines/<id>/spawn` 回 **404 Route not found**，所以 runner 對話的並不是這條路由／這個版本。
- `relay/src/index.ts` 自己就會回 `502 {"ok":false,"code":"upstream"}`；edge／relay 產生 502、
  而 origin 其實回了別的內容，是尚未排除的假設。
- 用 `~/.hapi/settings.json` 的 `cliApiToken` 去對公開 hub `/api/auth` 驗證會被拒
  （`Invalid access token`），但 `hapi` CLI 本身正常 → **不要用那個檔的 token 做探測**，會得到誤導的 401/404。

上游 issue [#2409](https://github.com/tiann/hapi/issues/2409)；維護版 fork 上另開了只補文件的
PR #2410（它主張上游本來就端到端傳遞 runner 的 message、502 來自 fork 帶的未合併 #1771，
但這個說法當時無法用現場證據證實）。重查前先確認真正的 hub 在哪、502 是 origin 還是邊緣層產生。

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
