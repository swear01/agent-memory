---
title: swairM5 全域 bun 已收成 Homebrew core，勿再 prepend ~/.bun/bin
scope: machines/swairM5
machine: swairM5
tool: bun
status: active
confidence: high
created: 2026-09-12
updated: 2026-09-12
tags:
  - bun
  - homebrew
  - PATH
---

# 現況

2026-09-12 已拿掉官方 installer 的 `<mac-home>/.bun/bin/bun`，並刪除 zsh 裡 `BUN_INSTALL` / `PATH=$BUN_INSTALL/bin` 的 prepend。全域 `bun` 只走 Homebrew core：`<homebrew-prefix>/bin/bun`。

當日驗證：`bun --version` 為 `1.4.0+1381054db`。Homebrew core formula 當時是 `1.4.0`；官方 GitHub latest 是 `1.4.2`，所以 brew 升級不會自動對上 oven-sh 最新 patch。之後用 `brew upgrade bun`。

`<mac-home>/.bun/install/cache` 仍保留，那是 bun 套件快取，不是 runtime binary。不要再把 `<mac-home>/.bun/bin` 加回 PATH，否則會再度蓋過 Homebrew。
