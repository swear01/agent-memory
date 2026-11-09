---
title: SwairM5 NTU ADFP VPN（adfp-vpn／openfortivpn）
scope: machines/swairM5
status: active
created: 2026-10-20
updated: 2026-11-09
tags: [ADFP, VPN, openfortivpn, FortiGate, Guacamole, sudoers, macOS]
---

# NTU ADFP VPN 與 Guacamole 登入：adfp-vpn

搜尋關鍵字：ADFP VPN、Guacamole 登入、NTU-ADFP-Workstation、FortiClient、openfortivpn、adfp-vpn、10.32.2.154。

2026-10-20 起 SwairM5 以 `adfp-vpn up|down|status` 連 ADFP，FortiClient 已解除安裝。
用法與故障處理見 `adfp-vpn` skill；安裝腳本在該 skill 的 `scripts/`。

- NTU-ADFP-Workstation 是 FortiGate SSL VPN（`140.112.33.153:10443`，出廠自簽憑證，
  無 OTP／SAML／EMS），openfortivpn 以 `trusted-cert` 固定憑證即可連線。split tunnel
  只導 `10.32.2.154/32`。工作站擋 ICMP，用 TCP 443 判斷可達性。
- 密碼只在 root-only `/etc/adfp-vpn/config`；sudoers 只開放 helper 的 up/down/status/login/unlock。
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

## ADFP 鎖屏（2026-10-25）

- 閒置後 xrdp 桌面會跳出 xfce 鎖屏；此時送出的按鍵會進密碼欄（曾誤送一行指令，顯示 Incorrect password）。
  打字前先截圖確認畫面。
- 2026-10-26 已驗證的解法：黑畫面先按 Space 喚醒並檢查鎖屏，再執行 `adfp-vpn unlock`；
  只有重新截圖確認 XFCE 桌面已解鎖後才繼續。受保護的 helper 代輸密碼，不自行讀取或填入密碼。
  重新載入可能因 session affinity 回到同一個鎖屏，不能當成可靠的解鎖方法。
- 使用者同意後已在共用家目錄關閉鎖屏：`xfce4-screensaver` 的 `/saver/enabled`、`/lock/enabled`、
  `/lock/saver-activation/enabled` 與 `xfce4-power-manager` 的 `dpms-enabled`、`lock-screen-suspend-hibernate`
  皆為 false。auto mode 分類器會把這類設定判為削弱安全，需使用者明確同意。

## Guacamole 單一使用者連線上限（2026-11-05 恢復驗證）

- 遇到「已超出單一使用者可同時使用的連線數量」（`TUNNEL_ERROR`、`statusCode 797`），
  先確認 `adfp-vpn status`。本次 VPN 正常，pool 的 `max-connections-per-user` 為 1。
- 在目前 Guacamole 頁面正常「登出」後，再執行 `adfp-vpn login`。本次使用頁面自己的
  `authenticationService.logout()`（此站呼叫 `DELETE api/session`）正常登出，重新登入後狀態
  由 `WAITING` 變為 `CONNECTED`，並實際看到 XFCE 桌面；沒有重啟 VM 或使用桌面 Force Logout。
- 只重新登入取得新 token、重載頁面或等待，不等同正常登出舊登入 session。token 應留在頁面，
  不要輸出或寫入記憶。若已失去舊登入 session，登出目前 session 不保證能釋放舊連線。
- pool 的 `activeConnections` 是全群組總數，不能當成此使用者占用數；活動連線 API 回 `{}`
  也不足以確定權限問題。這次確認了恢復方法，未確認舊連線來源或伺服器根因，不能斷言只能等逾時。
- 2026-11-09 再驗證（VPN 斷線重連後出現上限錯誤）：一行 CDP 刪掉目前 token，再 `adfp-vpn login`，
  約 30 秒回到 XFCE 桌面；不用等 60 秒、不用反覆按「重新連接」。
  `(async()=>{const s=angular.element(document.body).injector().get("authenticationService");`
  `return (await fetch(new URL("api/tokens/"+encodeURIComponent(s.getCurrentToken()),document.baseURI),{method:"DELETE"})).status})()`
  回 204 後執行 `adfp-vpn login`。token 在 localStorage `GUAC_AUTH_TOKEN`，不是 `GUAC_AUTH`。
- 一般使用者逐筆關連線不可行：`mysql`、`mysql-shared`、`ldap` 三個 data source 的
  `activeConnections` 都回 0 筆（即使正連著），所以無法 DELETE 單一舊連線。
- English keywords: Guacamole "exceeded connection limit" / max connections per user / stale tunnel after VPN drop /
  reconnect refused → logout token (DELETE api/tokens or api/session) + `adfp-vpn login`.

## 不用 Playwright 的 ADFP 操作（2026-10-25 驗證）

- 用 loopback CDP 9223 直接操作 ADFP 分頁：`Input.dispatchMouseEvent`（CSS 座標 = 截圖像素 ÷ 2）、
  `Input.dispatchKeyEvent` 逐字打字；Guacamole 選單用 Ctrl+Alt+Shift 開關。
- 上傳：先在頁面攔截 `HTMLInputElement.prototype.click`（file input 存到 `window.__fi`），選單→設備
  `/share1/home/<user>`→「上传文件」，再 `Runtime.evaluate('window.__fi')` 取 objectId，
  `DOM.setFileInputFiles({files, objectId})` 即上傳到家目錄（自動觸發 change）。macOS tar 先設
  `COPYFILE_DISABLE=1` 免 xattr 警告。
- 新開 terminal 是 tcsh，`dc_shell`、`vcs` 不在 PATH；先打 `cb`（Cell-Based）載入環境，再開 bash。
- CVSD HW3 SRAM（`sram_512x8` 等）：DC 2026.03 的 `write_sdf` 會寫出 SRAM 的 `PERIOD`（512×8 為 2.213 ns），
  gate-level 模擬不會用模型預設的 3.0 ns。
