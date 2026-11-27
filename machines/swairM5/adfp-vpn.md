---
title: SwairM5 NTU ADFP VPN（adfp-vpn／openfortivpn）
scope: machines/swairM5
status: active
created: 2026-10-20
updated: 2026-11-09
tags: [ADFP, VPN, openfortivpn, FortiGate, Guacamole, sudoers, macOS]
---

# ADFP VPN 可用但 Guacamole 仍不通

- SwairM5 使用 `adfp-vpn` 技能與課程 repo 的 `tools/adfp-chromium.command`。FortiTray 和 `scutil --nc` 查舊 FortiClient 的方法已退役。依技能的 `adfp-vpn status`／launcher／`login`／`unlock` 邊界操作；不讀 token、密碼、root-only config，不手填或反覆試登入。保護 helper 與憑證 pin 升級照技能。
- 若 VPN 已通但 Guacamole 回 `TUNNEL_ERROR`／`statusCode 797`，檢查目前連線狀態與單人連線上限。在仍保有原 session 的 Guacamole 頁面正常登出，透過 `adfp-vpn login` 重新登入，再以 `CONNECTED` 與實際 XFCE 畫面驗證。只刷新頁面、取新 token 或等一下不等於正常登出。已失去原 session 時，不保證登出新 session 會釋放舊連線；不要把群組總連線／空 API 當成個人占用，也不要直接 Force Logout 或重開 VM。
- 桌面黑屏先喚醒並看新截圖。若只看到鎖屏，走技能的 protected unlock，看到桌面才送終端指令；重新載入可能回到同一鎖屏。工作站擋 ICMP，連通性查實際 Guacamole 服務，不能拿 ping 結果代替。
- 檔案上傳或桌面操作以 [課程已驗證入口](../../projects/ntu-cvsd-adfp/lab-execution-pitfalls.md) 為準。新 terminal 預設 tcsh，先跑 `cb` 載入課程環境再開 Bash。GUI 上傳／選檔的專案機制依目前 launcher 檢查，避免觸碰任何 browser credential state。
