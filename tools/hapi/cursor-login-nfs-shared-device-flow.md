---
title: Cursor Agent 登入：swear01 四台共用一次授權，用 NO_OPEN_BROWSER device flow
scope: tools/hapi
status: active
confidence: high
created: 2026-11-17
updated: 2026-11-17
tags:
  - cursor-agent
  - login
  - nfs
  - hapi
source_refs:
  - live:fleet-2026-11-17
---

# 現象

`cursor-agent update` 回 `[unauthenticated]`、`cursor-agent status` 顯示 `Logged in (unable to fetch user details)`，代表 `~/.config/cursor/auth.json` 的 token 已死。用官方安裝器 `curl -fsSL https://cursor.com/install | bash` 仍可更新版本（Windows 見 machines/swop/ssh-reachability）。

# 登入

- `cursor-agent logout`，再 `NO_OPEN_BROWSER=1 cursor-agent login`（遠端要 `ssh -tt`），它會印出 `https://cursor.com/loginDeepControl?challenge=...` 一次性網址，使用者在自己的瀏覽器核准即可。流程在背景等待，約 15 分鐘逾時。
- mazu、athena、cthulhu、valkyrie 的 `~/.config/cursor` 是同一份（在 mazu 放標記檔，其他三台立刻看到），一次登入涵蓋四台。zeus（swear02）、oracle、swop、Mac 各自獨立。
- 登入後用 `cursor-agent status` 加 `cursor-agent -p --model grok-4.7-low "say ok"` 驗證。
