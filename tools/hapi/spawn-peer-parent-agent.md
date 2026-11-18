---
title: spawn_peer 省略 agent 會開成 hub 預設的 Claude，不會繼承父 session
scope: tools/hapi
tool: hapi
status: active
confidence: high
created: 2026-11-18
updated: 2026-11-18
tags:
  - hapi
  - spawn-peer
  - cursor
  - claude
  - oauth
---

# 現象

2026-11-18，在 swever（Oracle runner）從一個 Cursor 父 session 用 MCP `spawn_peer` 連開四個 worker，參數裡沒有 `agent`。`inspect_peer` 四筆都是 `flavor: claude`。每一筆的第一則 agent 訊息都是：

`Failed to authenticate: OAuth session expired and could not be refreshed`

session 仍是 `active: true`、`lifecycle: running`，但沒有工具呼叫，也沒有 git 動作。同機 Claude Code 2.1.312 的 `claude auth status` 是 `loggedIn: false`、`authMethod: none`。

# 根因

`spawn_peer` 省略 `agent` 時，依工具說明從 hub 的 peerSpawnDefaults（Settings → General → Agents）解析，沒有設定就用 stock `claude`。父 session 的 flavor 不會被抄過去。父 session 是 Cursor 時，省略 `agent` 就是在開 Claude。

`hapi-session-control` 寫「新 session 用這個 session 的 agent；只有使用者要求不同 agent 時才傳 `agent`」。把後句讀成「不要傳 agent」會開錯 flavor。父 session 的 flavor 和 hub 預設不同時，要明確傳 `agent`。

權限模式是另一條軸：省略 `permissionMode` 的後果見 `session-permission-preference.md`。

# 做法

- 從 Cursor session 派工時，`spawn_peer` 傳 `agent: "cursor"`。
- spawn 之後用 `inspect_peer` 核對 `flavor`。
- 看到上面那句 OAuth 錯誤時，先跑 `claude auth status`。同一批 Claude session 再 ping 不會換 flavor。使用者要改用 Cursor 時，新開 session 並帶 `agent: "cursor"`。
- 2026-11-18 依此重開四個 worker（上游 PR 2342、2096、2099、1418），`spawn_peer` 都回了新的 session id。

# 範圍

這次只證實 swever 上的 Claude 登入是空的，以及省略 `agent` 會解析成 Claude。Claude 重新登入之後，省略 `agent` 仍會開 Claude，只是不再卡在這句 OAuth 錯誤。

這條只適用 HAPI 的 MCP `spawn_peer`。Cursor 自己的登入見 `cursor-login-nfs-shared-device-flow.md`；oracle 的 Cursor 認證與 mazu 那四台分開。
