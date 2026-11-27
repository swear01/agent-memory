---
title: Claude Code fleet 訂閱啟動入口
scope: tools/anthropic
status: active
updated: 2026-11-19
---

# 訂閱入口與更新方式

Claude Code 會優先採用繼承的 API 認證。保留真正 API 程式使用的 shell 金鑰，只在 Claude 子程序移除 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_BASE_URL`。

Canonical source 為 `<transfer_MAC-root>/stow/core/.local/bin/claude`，部署成 `~/.local/bin/claude`。入口執行：

```sh
exec /usr/bin/env -u ANTHROPIC_API_KEY -u ANTHROPIC_AUTH_TOKEN -u ANTHROPIC_BASE_URL "$HOME/.local/lib/claude-code" "$@"
```

`~/.local/lib/claude-code` 指向原 npm package 的 `bin/claude.exe`；npm prefix 的 `bin/claude` 也連回入口。Oracle 舊 `/usr/local/bin/claude` 同樣連回受管理帳號入口。這涵蓋標準命令與原 npm 命令路徑，不能只設互動 shell alias。

直接更新既有安裝並重接入口。native installer 可能另裝執行檔並移除 npm 套件。發現入口不再連到 Stow 時，從 npm 更新再重接：

```sh
npm install -g @anthropic-ai/claude-code@latest
mkdir -p "$HOME/.local/lib"
ln -sfn "$(npm root -g)/@anthropic-ai/claude-code/bin/claude.exe" "$HOME/.local/lib/claude-code"
ln -sfn "$HOME/.local/bin/claude" "$(npm prefix -g)/bin/claude"
claude --version
claude auth status
```

## 更新後入口被覆寫

- 在 Mac，`com.swear.claude-relink` LaunchAgent 監看 npm bin，修復 npm 自動更新重寫的 symlink。來源在 transfer_MAC `config/snapshot/home/Library/LaunchAgents/`。若 `claude update` 回 `Couldn't install the update`，也避免改跑 `claude install`。若 npm 遇 `EEXIST <npm-prefix>/bin/claude` 且確認是 relink 競態，可用 `npm install -g --force @anthropic-ai/claude-code@latest` 更新該既有 package，讀回 symlink 與認證。
- Claude Code 2.1.314 的 “Multiple installations found” 可能只因 npm 套件和 `~/.local/bin/claude` wrapper 同時存在。先核對 symlink 的真正 binary，勿直接 uninstall npm 套件。
- Linux 的 transfer_MAC `scripts/claude-relink.sh` 由本機 cron 定期把 nvm npm 的 bin 連回入口；Oracle 用 systemd user timer。共用 NFS 上的安裝只更新一次，relink 仍需逐台確認。別從多台主機同時更新同一份 npm 安裝。
- 自動更新要同時核對 wrapper 與 `~/.claude.json`。`DISABLE_UPDATES`、`DISABLE_AUTOUPDATER` 或 unprotected global 安裝的 `autoUpdates: false` 均會停用。2026-10-31 採用 Mazu 單機更新共用 home 的設計，其他 Linux wrapper 帶 `DISABLE_AUTOUPDATER=1`。Mac 用原設定。核對現行實際配置再更新。

## 驗證

沿標準入口與原 npm bin 驗證版本、symlink、`claude auth status`（應見 `claude.ai`，以及預期的帳戶方案）；HAPI 應另用 runner 的 PATH 核對。用已取得授權的實際請求驗證指定模型，不能將登入和版本讀回當成所有主機的請求驗證。更新 binary 後舊程序不會換版，SDK、直接 HTTP 與原 binary 不受 wrapper 認證清除影響。

歷次 fleet 明細、範圍與異常見 `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/anthropic/claude-subscription-launcher.md`。來源是 transfer_MAC PR #64 和 `docs/claude-subscription-fleet.md`。
