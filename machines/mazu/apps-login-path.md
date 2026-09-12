---
title: Mazu login PATH 需壓過 profile.d 的 /usr/local/bin
scope: machines/mazu
machine: mazu
status: active
created: 2026-09-12
updated: 2026-09-12
tags:
  - apps
  - login
  - path
  - profile.d
---

# 結果

- Task 5 在 Mazu 本機登入檔加入 `/apps/bin`：`/etc/profile.d/Z20-dvlab-apps.sh`（TMPDIR + source `/apps/etc/env.sh`）、`/etc/environment` PATH、`/etc/zsh/zshenv` emulate-sh source Z20。沒有 source 完整 `vcs.sh` / `verdi.sh` / `spyglass.sh`。
- 只改 `/etc/environment` 不夠。既有 `/etc/profile.d/dvlab-env.sh` 會把 `/opt/nvim/bin:/opt/maven/bin:/opt/miniconda/bin:/usr/local/bin` 插到 PATH 最前，bash login 仍會先命中舊的 `/usr/local/bin/vcs`。
- 因此多了一個較晚執行的 `/etc/profile.d/zz-dvlab-apps-path.sh`，在其他 profile.d 之後再 `export PATH="/apps/bin:$PATH"`。`run-parts` 順序上它在 `dvlab-env.sh` 之後。使用者 `~/.profile` 仍可能把 `~/.local/bin` 等插更前面，只要 `/apps/bin` 在 `/usr/local/bin` 之前即可。
- fish 可省略：launcher 自己 source license；PATH 由 PAM `/etc/environment` 處理。
- 不要印 license 值，不要 cat `license.sh`。Login 後 `VCS_HOME` 應為空，PATH 不應出現 `/apps/cad/...` tool bin。

# 驗證帳號

- Brief 的 `getent passwd | awk UID>=1000 && !=swear02` 在 Mazu 會先選到 `nobody`（65534、`nologin`、home `/nonexistent`），`sudo -u nobody -i` 失敗。改選有 login shell 的 NIS 使用者。
- NIS `getent passwd` 可能回傳 password hash，禁止把整筆 record 印出。細節見 `machines/hapi-fleet/nis-account-query-safety.md`。
