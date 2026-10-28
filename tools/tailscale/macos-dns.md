---
title: macOS 上 Tailscale DNS 排查
scope: tools/tailscale
status: active
updated: 2026-10-28
tags:
  - tailscale
  - dns
  - macos
---

# macOS 上 Tailscale DNS 排查

## M5 DNS 異常與 Tailscale 半升級（2026-10-28）
- 症狀：早上 8–9 點 DNS 解析失敗（網路通但網頁打不開）。系統 log 已被清掉，根因無法證實。
- 嫌疑：Tailscale「Use Tailscale DNS」開啟，所有查詢先走 100.100.100.100 再轉 1.1.1.1；轉發卡住就全滅。
- 發現：App 1.104.2，但 network extension 仍是 1.102.4（`systemextensionsctl list | grep -i tailscale`，CLI 也會警告 version mismatch）。
- 修法：`osascript -e 'quit app "Tailscale"'; open -a Tailscale`，extension 自動換成新版，舊版標記 reboot 時移除。
- 驗證：`dscacheutil -q host -a name google.com` 與 `... swever.tailbfa9de.ts.net` 都要能解析。
- 禁止：不要用 `tailscale set --accept-dns=false`，使用者需要 .ts.net 名稱解析。
- 下次再發生：當下立刻抓 `scutil --dns`、`tailscale dns status`、`log show --last 10m --predicate 'process=="mDNSResponder"'`。
