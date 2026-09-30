---
title: Fleet Codex 與 Pi 的安裝位置與更新方式
scope: tools/hapi
tool: hapi
status: active
updated: 2026-09-30
---

# 結論

Fleet 的 Codex / Pi 是各 runner 使用者的 npm global，不是 system-wide。更新時只寫獨立 home，NFS 四台只寫一次。不要重啟 Runner；既有 session 進程會繼續用舊 binary。

# 2026-09-11 已驗證版本

目標：`@openai/codex@0.154.0`、`@earendil-works/pi-coding-agent@0.85.1`。

已核對：mazu、cthulhu、athena、valkyrie、zeus、oracle、Mac。Windows `swop` 當日不在 Hub machine 列表，舊區網位址也不通，未更新。

# 2026-09-30 Codex 0.159.2 更新

八台（含 Windows swop）皆已驗證 Codex `0.159.2`；Pi 不在本次更新範圍。模型 catalog 與推論邊界見 `tools/hapi/codex-model-catalog-allowlist.md`。

NFS 共用 home 原地 `npm install -g --prefix <nvm-node-dir>` 遇到 `EBUSY`：npm 嘗試移除仍被執行中程序使用的 `.nfs*` binary，並已讓原命令入口失效。不能只記錄 install 失敗後離開，也不能為更新而殺掉工作中的程序。

已驗證的恢復方式是以 npm 安裝到新的使用者版本 prefix `~/.local/codex-0.159.2`，確認該 prefix 的 `bin/codex --version` 成功後，再切換 `~/.local/bin/codex` symlink。NFS `swear01` 共用入口只寫一次，mazu、athena、cthulhu、valkyrie 分別驗證；該 home 原 nvm 的 `bin/codex` 也已改指向新版入口，仍被程序持有的舊 `.nfs*` 保留。

Zeus `swear02` 與 Oracle `ubuntu` 同樣採各自的 `~/.local/codex-0.159.2` npm prefix，`~/.local/bin/codex` 指向新版。Mac 沿用 `~/.npm-global`，Windows 沿用現有 WinGet Node 的 npm global prefix。下次升級先沿 `command -v`、symlink 與 package ownership 找到實際入口，不能假設 shell 當下的 `npm prefix -g` 就是使用中的安裝位置。

Windows PowerShell 的 npm／Codex `.ps1` 入口可能受 execution policy 限制；本次使用 `npm.cmd` 與 `codex.cmd`。Python updater 的 `subprocess` 版本探測以已安裝套件內實際 `codex.exe` 的目錄暫時置於 PATH 前方成功，不需永久修改系統設定。自製驗證腳本讀取 UTF-8 catalog 時須明確指定 `encoding="utf-8"`，避免 Windows 預設 cp950 解碼失敗。

# 2026-09-11 各 home 入口快照

- NFS `swear01`（mazu / cthulhu / athena / valkyrie）：nvm `v24.15.0` 的 `npm prefix -g`。`~/.local/bin/codex` 連到該 nvm `codex`；`~/.local/bin/pi` 連到 `pi-safe`。Runner `PATH` 以 `~/.local/bin` 再接 nvm bin。只在其中一台執行 `npm install -g`。
- Zeus `swear02`：同樣走 nvm `v24.15.0`。沒有 `~/.local/bin/codex`，Runner 靠 nvm bin；`pi` 仍經 `pi-safe`。
- Oracle `ubuntu`：live Codex 在 nvm `v24.15.0/lib/node_modules/@openai/codex`，由 `~/.local/bin/codex` 連過去。該 nvm 的 `npm prefix -g` 卻是 `/usr/local`，直接 `npm install -g` 會打到系統舊副本（曾見 `/usr/local` 的 `0.144.1`）。必須 `npm install -g --prefix <nvm-node-dir>`。live Pi 在 `~/.local/lib/node_modules/@earendil-works/pi-coding-agent`，由 `pi-safe` 直接 exec `dist/bundle/cli.js`。PM2 `PATH` 以 `~/.local/bin` 為首。
- Mac：`~/.npm-global`。LaunchAgent `PATH` 含 `~/.local/bin` 與 `~/.npm-global/bin`。`pi` 經 `pi-safe` 指向 npm-global 的 `pi`。
- Windows `swop`：Node 可能在 WinGet 版號路徑，不在預設 PATH。先用絕對路徑找現有 `npm` / `codex` / `pi`，不要重裝 Node。

# 更新指令

官方穩定版：

```bash
npm install -g @openai/codex@<version>
npm install -g --ignore-scripts @earendil-works/pi-coding-agent@<version>
```

Pi 必須加 `--ignore-scripts`。不要安裝 GitHub 上的 `0.155.0-alpha.*` 除非操作者明確要求。
