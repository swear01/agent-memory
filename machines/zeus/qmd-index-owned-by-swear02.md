---
title: Zeus 的 QMD index 屬於 swear02，要用 su_zeus 更新
scope: machines/zeus
machine: zeus
status: active
created: 2026-10-22
updated: 2026-10-22
tags:
  - qmd
  - permissions
---

- `/var/tmp/qmd-memory-zeus` 由 swear02 擁有（775），collection 指向 swear02 家目錄下的 `~/agent-memory`（swear02 自己的 checkout，不是 swear01 的 NFS home）。
- 用 `ssh zeus`（swear01）跑 `qmd update` 會報 `SQLITE_READONLY_DIRECTORY`；swear01 沒有 sudo，不能 chown。
- 正確做法：`ssh su_zeus`（swear02），在 `~/agent-memory` 以標準 QMD env 前綴跑 `qmd update`、`qmd embed`、`qmd search`。2026-10-22 這樣更新成功。
- swear01 在 zeus 上的 `~/agent-memory` 是 valkyrie/cthulhu/mazu 共用的 NFS checkout，用 git pull 同步即可，不需要 zeus 專屬的 index。
