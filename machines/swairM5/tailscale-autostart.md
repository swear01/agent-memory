---
title: M5 的 Tailscale 開機自啟動與連 oracle 的 SSH
scope: machines/swairM5
tags: [tailscale, macos, login-item, oracle, ssh]
status: active
created: 2026-11-21
updated: 2026-11-21
---

2026-11-20 救援 oracle 時發現 M5 的 Tailscale 是停的（`tailscale status` 顯示 "Tailscale is stopped"），所以走 tailnet 的 SSH 一直 timeout。

- 原因：Tailscale 主程式沒在跑，只有 network extension 在背景。它的 `TailscaleStartOnLogin = 0`，macOS 登入項目裡也沒有 Tailscale。
- 已處理：
  - 用 `osascript -e 'tell application "System Events" to make login item at end with properties {path:"/Applications/Tailscale.app", hidden:true}'` 把它加進登入項目。
  - `defaults write io.tailscale.ipn.macsys TailscaleStartOnLogin -bool true`。
  - `open -g -j -a Tailscale` 啟動。
  - 執行 `tailscale up` 後恢復連線。
  - 「重開機後會自動連上」這點還沒實測。
- 版本不一致：重新開啟 App 後，network extension 從 1.104.2 換成 1.104.4，舊版標記成重開機時移除，和 [[macos-dns]] 記錄的模式相同。
- 從 M5 走 tailnet SSH 到 oracle：`ssh ubuntu@100.126.43.9`。第一次會出現 `Host key verification failed`，因為 known_hosts 裡沒有這個 IP。先確認 `ssh-keyscan -t ed25519 100.126.43.9` 的指紋和 `ssh-keygen -F 138.2.52.9` 的相同，再加進 known_hosts。

相關：[[ubuntu-2604-upgrade]]（oracle 的 SSH 救援路徑）。
