---
title: swairM5 全域 bun 被 ~/.bun 蓋過 Homebrew，兩邊都可能停在舊版
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

# 安裝來源

swairM5 同時有兩份 bun：

- 官方安裝器：`<mac-home>/.bun/bin/bun`。zsh 用 `BUN_INSTALL` 把它 prepend 到 `PATH`，所以 `which bun` 永遠先命中這裡。
- Homebrew core：`<homebrew-prefix>/opt/bun/bin/bun`。`brew info bun` 會警告它被前面那份 shadow。

2026-09-12 查核時，兩邊都是 `1.3.14+0d9b296af`（2026-05-13），官方最新是 `1.4.2`，Homebrew core formula 只到 `1.4.0` 且未 upgrade。先前 HAPI `v0.29.1.1` CI 用 Bun 1.4.0 跑 frozen lockfile，本機 1.3.14 生的 `bun.lock` 會被拒絕。

# 升級要注意

只跑 `brew upgrade bun` 不會改 `bun --version`，因為 PATH 仍指向官方安裝器。要更新實際在用的那份，對 `<mac-home>/.bun/bin/bun` 跑 `bun upgrade`。Homebrew 那份另外用 `brew upgrade bun`；core formula 可能仍落後官方 GitHub release。
