---
title: SwairM5 的 NTU DGX VPN 與 ADFP 並行
scope: machines/swairM5
status: active
updated: 2026-11-11
---

- 「DGX VPN」與「ADFP VPN」是 macOS 應用程式資料夾內兩個獨立捷徑，共用 `/usr/local/libexec/ntu-vpn/toggle`。點一次切換；CLI 分別為 `dgx-vpn up|down|status`、`adfp-vpn up|down|status`。
- DGX 仿 ADFP 用 root-owned openfortivpn、root-only `/etc/dgx-vpn/config` 和只允許三個命令的 sudoers 規則。Vaultwarden 在安裝時提供 VPN 密碼，執行時直接由 root 設定讀取，不需要輸入。SSH 密碼不在 VPN 設定裡。
- ADFP helper 依可執行檔路徑找行程，DGX 因此有獨立的 `/usr/local/libexec/dgx-vpn/openfortivpn`、PID 與路由。DGX 只導向 `192.168.163.102/32`，不改預設路由或 DNS。OpenSSL 依賴沿用 ADFP 已安裝的 root-owned 複本；更新 ADFP 的 OpenSSL 或重新安裝時，應一起檢查 DGX。
- 2026-11-11 用正式切換程式驗證：兩條 VPN 同時開，ADFP TCP 443 可達，DGX SSH 成功登入 `DGXH100`（Linux）。當時 ADFP 用 `ppp0`、DGX 用 `ppp1`，預設路由仍用 `en0`；PPP 編號可能變動。關掉 DGX 後，ADFP TCP 443 仍可達。
