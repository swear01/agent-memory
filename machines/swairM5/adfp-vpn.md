---
title: SwairM5 NTU ADFP VPN（adfp-vpn／openfortivpn）
scope: machines/swairM5
status: active
created: 2026-10-20
updated: 2026-10-20
tags: [ADFP, VPN, openfortivpn, FortiGate, Guacamole, sudoers, macOS]
---

# NTU ADFP VPN 與 Guacamole 登入：adfp-vpn

搜尋關鍵字：ADFP VPN、Guacamole 登入、NTU-ADFP-Workstation、FortiClient、openfortivpn、adfp-vpn、10.32.2.154。

2026-10-20 起 SwairM5 以 `adfp-vpn up|down|status` 連 ADFP，FortiClient 已解除安裝。
用法與故障處理見 `adfp-vpn` skill；安裝腳本在該 skill 的 `scripts/`。

- NTU-ADFP-Workstation 是 FortiGate SSL VPN（`140.112.33.153:10443`，出廠自簽憑證，
  無 OTP／SAML／EMS），openfortivpn 以 `trusted-cert` 固定憑證即可連線。split tunnel
  只導 `10.32.2.154/32`。工作站擋 ICMP，用 TCP 443 判斷可達性。
- 密碼只在 root-only `/etc/adfp-vpn/config`；sudoers 只開放 helper 的 up/down/status/login。
  openfortivpn 與 Homebrew OpenSSL 複製到 root-owned 目錄並改連結，避免同使用者改寫
  `/opt/homebrew` 後劫持 root 程序；brew 升級後要重跑安裝腳本才會換版。
- Guacamole（`https://10.32.2.154/`）只要求 username／password（無 TOTP／SSO）。
  `adfp-vpn login` 由 root helper 讀 root-only `/etc/adfp-vpn/guacamole`，以憑證
  SHA-256 pin 呼叫 `/api/tokens`，再經 loopback CDP 9223 把 token 寫進 ADFP 分頁的
  localStorage `GUAC_AUTH_TOKEN`（Guacamole 存的是 `JSON.stringify(token)`）並重新整理。
  課程 repo 的 `tools/adfp-chromium.command` 開瀏覽器後會自動執行；session 過期出現
  登入頁時再跑一次即可，agent 不應自行填登入表單。
- 不要改放 login keychain：同使用者的任何 agent 都能用 `security find-generic-password -w` 讀出。
- 舊做法（FortiTray 選單連線、`scutil --nc status` 查 FortiClient 服務）已失效。
  課程 repo PR #14（2026-10-20 合併，`51fa23d`）已把啟動器改為 `adfp-vpn up` 加自動
  `adfp-vpn login`；Swear Review 在該 repo 兩次回報 `no items were selected`（未實際審查），
  不能當通過。
