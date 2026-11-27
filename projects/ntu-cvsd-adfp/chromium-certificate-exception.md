---
title: ADFP dedicated Chromium SPKI exception
scope: projects/ntu-cvsd-adfp
status: launcher-verified; ADFP-live-test-blocked-by-VPN
updated: 2026-10-20
---

- 自簽憑證缺 SAN 時，匯入 Keychain 不能補 hostname 檢查。2026-10-13 使用者已核准 ADFP 專用 Chromium profile 的固定 SPKI，私有 CVSD repo 的 `tools/adfp-chromium.command` 使用 `--ignore-certificate-errors-spki-list`。用該既有 launcher 的核准 key，勿以未驗證連線抓到的 key 自動替換。
- 匹配 SPKI 的 chain 可略過 hostname 等憑證錯誤，涵蓋範圍比 exact leaf fingerprint 大。僅在 ADFP 專用 profile 啟用，直接開一般 Chromium 不會保留 startup flag。移除 flag 後重新啟動即可撤回；server 換 key 要重新核驗信任。
- Chromium 150 本機實驗中匹配 key 可載入，無匹配則失敗，核准的 ADFP key 也拒絕無關測試憑證。曾讀回專用 Chromium 啟動參數，當輪未核實 ADFP 線上無警告。
- VPN 已改用 `adfp-vpn`，舊 FortiClient recovery 不再適用。完整 `scutil --nc status` 曾可能含 VPN session secret；診斷只輸出必要狀態行。相關 [[machines/swairM5/adfp-vpn]]。
