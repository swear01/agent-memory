---
title: Claude Code fleet 訂閱啟動入口
scope: tools/anthropic
status: active
updated: 2026-10-31
---

# 訂閱入口與更新方式

Claude Code 會優先採用繼承的 API 認證。保留真正 API 程式使用的 shell 金鑰，只在 Claude 子程序移除 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`ANTHROPIC_BASE_URL`。

Canonical source 為 `<transfer_MAC-root>/stow/core/.local/bin/claude`，部署成 `~/.local/bin/claude`。入口執行：

```sh
exec /usr/bin/env -u ANTHROPIC_API_KEY -u ANTHROPIC_AUTH_TOKEN -u ANTHROPIC_BASE_URL "$HOME/.local/lib/claude-code" "$@"
```

`~/.local/lib/claude-code` 指向原 npm package 的 `bin/claude.exe`；npm prefix 的 `bin/claude` 也連回入口。Oracle 舊 `/usr/local/bin/claude` 同樣連回受管理帳號入口。這涵蓋標準命令與原 npm 命令路徑，不能只設互動 shell alias。

使用者要求直接更新既有安裝；這次不保留另一套版本目錄。2026-10-21 起依使用者要求移除 `DISABLE_UPDATES=1`，讓 Claude 自動更新；代價是自動更新或 native installer 可能改寫 `~/.local/bin/claude`，使 API key 排除失效。發現入口不再連到 Stow 時，從 npm 更新再重接：

```sh
npm install -g @anthropic-ai/claude-code@latest
mkdir -p "$HOME/.local/lib"
ln -sfn "$(npm root -g)/@anthropic-ai/claude-code/bin/claude.exe" "$HOME/.local/lib/claude-code"
ln -sfn "$HOME/.local/bin/claude" "$(npm prefix -g)/bin/claude"
claude --version
claude auth status
```

# 2026-10-17 已驗證範圍

- Mac、Mazu、Cthulhu、Valkyrie、Zeus、Oracle 原安裝為 `2.1.302`。用假 API key、token、endpoint 測一般入口與原 npm 命令路徑，六台皆為 `claude.ai` / `pro`，沒有 API key 來源；實際 HAPI Runner 的 PATH 也解析到入口。
- Mac、Mazu、Zeus、Oracle 實際 `--model opus` 請求使用 `claude-opus-5-5`。其餘兩台只驗證入口、版本與認證；不要擴大為六台皆完成模型請求。
- Zeus、Oracle 由使用者本人 `claude auth login --claudeai` 重新登入後驗證成功；未複製其他機器憑證。Oracle 先前未登入的結果已過時。
- Mazu／Cthulhu／Valkyrie／Athena 共用 home，安裝只從 Mazu 更新一次。Athena 已送修，依[排除規則](../../machines/athena/repair-status.md)不連線；共用磁碟更新不代表 Athena runtime 已驗證。Windows swop 連線不可用，使用者指示略過，未更新。
- 原程序不會隨磁碟更新切換版本，需重開。直接 HTTP／SDK、套件內 binary 或 settings 重新注入認證仍可繞過入口；這不是全域 API key 封鎖。

來源：transfer_MAC PR #64（merge `8460f976bce1e9c192f46d4acebd24c5afd94c65`），120 項測試與 CI 通過；完整部署記錄位於該 repo `docs/claude-subscription-fleet.md`。

# 2026-10-31 Mac 的 npm 入口被自動更新覆寫與 "Multiple installations" 警告

- Claude 自動更新用 npm 重裝，會把 `<npm-prefix>/bin/claude` 重寫成直接指向 `claude.exe`，跳過入口，API key 排除失效。Mac 已加 LaunchAgent `com.swear.claude-relink`（`WatchPaths` 監看 `~/.npm-global/bin`，已連對時為 no-op；實測手動改壞後 12 秒內連回）。來源 plist 在 transfer_MAC `config/snapshot/home/Library/LaunchAgents/`。其他機器沒有此保護，仍靠手動重接。
- `claude update`／`/doctor` 的 “Multiple installations found” 是誤報：2.1.314 的偵測只看 `<npm-prefix>/bin/claude` 加 npm 套件存在，且 `~/.local/bin/claude` 存在就標 "native"，沒有設定可關。`~/.local/bin/claude` 其實是入口 wrapper，真正的程式只有 npm 那份；不要照警告 `npm uninstall -g @anthropic-ai/claude-code`，會讓入口失效。要消除警告只能把入口搬離 `~/.local/bin`，影響整個機群 PATH，判斷不值得。
