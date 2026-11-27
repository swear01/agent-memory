---
title: Fleet Codex 與 Pi 的安裝位置與更新方式
scope: tools/hapi
tool: hapi
status: active
updated: 2026-10-28
---

## 沿 runner 的實際入口查安裝

Codex 與 Pi 是 runner 使用者的 npm 安裝。先沿 runner PATH、`command -v`、symlink 與套件 ownership 查實際入口。共享 NFS home 只更改一次，共用它的 runner 分別驗證；獨立 home 各自處理。保留執行中 session 與原 binary，完成後用新程序驗證，不必為更新而重啟 Runner。

- **NFS 原地更新遇 `EBUSY` 或入口已失效。** npm 刪除仍被程序使用的 `.nfs*` 會失敗。裝至使用者版本 prefix（例 `~/.local/codex-<version>`），先用該 prefix 的 `bin/codex --version` 驗，再切 runner 使用的 symlink，必要時也重接 nvm bin。保留舊 `.nfs*` 不殺使用中的程序。
- **Oracle 的 `npm prefix -g` 與 live PATH 不一致。** 曾經顯示 `/usr/local`，但實際 runner 用 nvm／`~/.local/bin`。明確指定正確 prefix。Pi 也可能由 `pi-safe` 直接 exec npm 套件的 `dist/bundle/cli.js`。Mac 的 prefix 當時為 `~/.npm-global`。
- **Windows execution policy 阻止 npm／Codex `.ps1`。** 查 WinGet Node 及原安裝，不重裝 Node。用 `npm.cmd`／`codex.cmd`，subprocess 可暫時將現有 `codex.exe` 的目錄加在 PATH 首。讀 model catalog 明確用 UTF-8，避免預設 cp950。
- 安裝已選定的 Codex 版本用 `npm install -g @openai/codex@<version>`，Pi 用 `npm install -g --ignore-scripts @earendil-works/pi-coding-agent@<version>`。實際 model catalog、登入與推論需分開讀回。歷史部署批次、prefix 快照與登入結果見 `git show 9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e:tools/hapi/fleet-codex-pi-install.md`。
