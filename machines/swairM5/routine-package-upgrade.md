---
title: swairM5 例行套件升級的坑（npm 11 install scripts、sudo cask、pi 模型清單）
scope: machines/swairM5
machine: swairM5
tool: homebrew, npm, mas, pi
status: active
confidence: high
evidence: 2026-11-19 季度升級實測（brew formula/cask、npm global、uv tool、skillshare CLI、pi 0.87.1→1.5.2）。
created: 2026-11-19
updated: 2026-11-19
tags:
  - upgrade
  - npm
  - homebrew
  - pi
---

# 例行升級注意事項

- **npm 11 預設跳過 install scripts**：`npm install -g` 只印 `npm warn install-scripts`，`node-pty`、`koffi`、`esbuild`、`node-llama-cpp`、`playwright-chromium` 等不會建置。照警告列出的名單重跑 `npm install -g --allow-scripts=<pkg,...> <pkgs>`。
- **claude-code 升級**：`~/.npm-global/bin/claude` 是連到 `~/.local/bin/claude` launcher 的 symlink，npm 會報 EEXIST。先 `rm` 掉這個 symlink，裝好後 `ln -sfn ~/.local/bin/claude ~/.npm-global/bin/claude`。
- **需要 sudo 的項目 agent 跑不了**：`mas upgrade`，以及含 pkg 或 sudo 的 cask（cyberduck、docker-desktop、google-drive、karabiner-elements、microsoft-office、obs、steam、tailscale-app、tunnelblick、xquartz、zoom）。把它們整理成一行 `sudo -v && mas upgrade && brew upgrade --cask --greedy ...` 交給使用者。
- **pi 的 `--list-models` 只列 2 個 DeepSeek 不算 regression**：`~/.local/bin/pi` 是 `pi-safe` wrapper，會過濾 stdout。驗證要用 RPC：`{"id":"1","type":"get_available_models"}` 透過 `--mode rpc --no-session` 送出，應有 9 個模型。預設的 `valkyrie-ninfer/qwen3.8-27b` 走 xhigh thinking，`pi -p` 90 秒內常常回不來，冒煙測試改用 `--model opencode-go/deepseek-flash`。pi 1.5.2 已驗證可用（RPC 清單相同，Codex 推論也正常）。
- **block-goose cask 有 receipt，但 `/Applications/Goose.app` 已經不在**，所以 upgrade 會失敗。CLI 改用 `block-goose-cli` formula。
- Swear Review 遇到只改 docs 的 PR 會顯示 "Review skipped: no items were selected"，這不算真的通過，要改用 `/gemini review`。

相關：[[bun-dual-install-path]]、[[fleet-cli-and-skill-upgrade]]
