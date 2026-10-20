---
title: SwairM5 NTU ADFP VPN（adfp-vpn／openfortivpn）
scope: machines/swairM5
status: active
created: 2026-10-20
updated: 2026-10-20
tags: [ADFP, VPN, openfortivpn, FortiGate, sudoers, macOS]
---

# NTU ADFP VPN：adfp-vpn

搜尋關鍵字：ADFP VPN、NTU-ADFP-Workstation、FortiClient、openfortivpn、adfp-vpn、10.32.2.154。

2026-10-20 起 SwairM5 以 `adfp-vpn up|down|status` 連 ADFP，FortiClient 已解除安裝。
用法與故障處理見 `adfp-vpn` skill；安裝腳本在該 skill 的 `scripts/`。

- NTU-ADFP-Workstation 是 FortiGate SSL VPN（`140.112.33.153:10443`，出廠自簽憑證，
  無 OTP／SAML／EMS），openfortivpn 以 `trusted-cert` 固定憑證即可連線。split tunnel
  只導 `10.32.2.154/32`。工作站擋 ICMP，用 TCP 443 判斷可達性。
- 密碼只在 root-only `/etc/adfp-vpn/config`；sudoers 只開放 helper 的 up/down/status。
  openfortivpn 與 Homebrew OpenSSL 複製到 root-owned 目錄並改連結，避免同使用者改寫
  `/opt/homebrew` 後劫持 root 程序；brew 升級後要重跑安裝腳本才會換版。
- 不要改放 login keychain：同使用者的任何 agent 都能用 `security find-generic-password -w` 讀出。
- 舊做法（FortiTray 選單連線、`scutil --nc status` 查 FortiClient 服務）已失效；
  依賴它們的腳本需改呼叫 `adfp-vpn`。
