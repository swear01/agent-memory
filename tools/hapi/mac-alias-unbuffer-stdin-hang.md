---
title: Mac 上 hapi alias 包了 unbuffer，--message-file - 會永遠卡住
scope: tools/hapi
tool: hapi
machine: mac
status: active
confidence: high
created: 2026-11-12
updated: 2026-11-12
tags:
  - hapi
  - ping-peer
  - spawn-peer
  - stdin
  - unbuffer
---

# 現象

2026-11-12 在 Mac 用 `hapi ping-peer <id> --message-file - --json <<'EOF' ... EOF` 送交接訊息，
跑了兩個多小時沒結束，輸出是空的，對方 session 也沒收到。

# 根因

Mac 的 `~/.zshrc` 有 `alias hapi='unbuffer $HOME/.local/bin/hapi'`。Agent 的 Bash tool 也會載入這個 alias。
`unbuffer`（沒加 `-p`）會把子程序的 stdin 換成 pty，所以 heredoc 或 pipe 的內容根本送不到 hapi。
hapi 的 fd 0/1/2 都指向 `/dev/ttysNNN`，也沒有任何 TCP 連線，就這樣一直等 stdin 的 EOF。
`spawn-peer-worktree-grouping.md` 已經記過「不要加 unbuffer」，但這次是 alias 自動加上的，從指令表面看不出來。

# 做法

- 要用 stdin 傳訊息（`--message-file -`）時，改用絕對路徑 `$HOME/.local/bin/hapi`，或寫成 `command hapi` / `\hapi` 繞過 alias。
- 也可以先把訊息寫成檔案，再用 `--message-file <path>`。
- 判斷方法：`type hapi`；卡住時用 `lsof -p <pid>` 看 fd 0 是不是 tty，有沒有 TCP 連線。
- 改用絕對路徑後，`ping-peer` 幾秒內就回傳 `{"ok":true,...,"remitId":...}`。
