---
title: SWear01_PC 的 Windows OpenSSH key-only 管理入口
scope: machine
machine: SWear01_PC
status: active
created: 2026-09-04
updated: 2026-09-05
tags:
  - windows
  - openssh
  - ssh
  - key-only
  - networking
---

# 已驗證狀態

- 主機為 Windows 11 25H2，使用 OpenSSH Server 9.5p2；`sshd` 為 Automatic、Running，並在內部 TCP 22 提供服務。
- Windows Administrators 群組依預設 `Match Group administrators` 規則使用 `C:\ProgramData\ssh\administrators_authorized_keys`。SwairM5 的 `~/.ssh/id_ed25519.pub` 已安裝；記憶中不保存 key 內容或 fingerprint。
- `administrators_authorized_keys` 的 ACL 只保留 `SYSTEM` 與 `BUILTIN\Administrators`。非 elevated `Get-Content` 出現 `Access denied` 是預期保護，不可為方便讀取而放寬 ACL；執行 hardening 與 `sshd -t` 必須 elevated。
- 生效硬化為 `PubkeyAuthentication yes`、`PasswordAuthentication no`、`AuthenticationMethods publickey`，並以 `AllowUsers` 限制單一管理帳號；記憶中不保存帳號名稱。
- SwairM5 經 LAN 使用 key 登入成功。client 停用 publickey 後，server 只宣告 `publickey`，回傳 `Permission denied (publickey)`，且沒有密碼提示。

# 網路邊界

- TP-Link Archer AX10 原廠官方 User Guide 只文件化 Web 管理與 Tether，未提供 SSH 或 Telnet CLI。
- LAN 端實機探測顯示 TCP 22、23 關閉，80、443 開放；目前原廠韌體沒有可用且受支援的 CLI。除非使用者另行明確要求，不要嘗試隱藏介面或第三方韌體。
- 家中網路為雙重 NAT；外層 DMZ 指向內層路由器、內層 port forwarding 是預期拓撲，但必須由真正的外部來源驗證。
- SwairM5 與 Windows 位於同一 LAN；從 SwairM5 直連 public IP 只是在測 NAT hairpin，不能證明外網可達。既有 Oracle ProxyJump 可作真外網來源。
- 2026-09-04 由 Oracle 真外網探測 public TCP 22 時立即得到 `Connection refused`，未進入 Windows SSH handshake；當時已將問題定位在 NAT/router path。
- 2026-09-05 實機確認外層 UGRID 閘道直接取得 public WAN、連線正常，DMZ 已啟用且目標吻合內層 Archer 的 WAN client，外層 firewall 為 Low；因此可排除 CGNAT 與外層 DMZ 指錯。
- 根因是內層 Archer 的 NAT forwarding 仍指向過期的 LAN 位址，其中 SSH 並未指向 Windows。SSH、7D2D 與 Minecraft 三筆既有規則已改指向 Windows 的目前位址，並於重新載入後確認保存。
- Archer 目前韌體的 forwarding 表單允許外部埠範圍，但內部埠欄只接受單一數值；範圍型規則應保留外部範圍，並將內部埠正規化為起始埠，否則表單會回報格式錯誤。
- 新增 DHCP reservation 時出現 `imb duplication`，原因是同一台 Windows 的 MAC 已有指向舊位址的 IP&MAC binding。該頁沒有編輯操作；安全修復是只刪除這一筆過期綁定，再以同一裝置的目前位址重建，並於重新載入後確認舊綁定已消失、新綁定已保存。
- 修復後由 Oracle 真外網確認 public TCP 22 已開啟；ProxyJump 到達 Windows OpenSSH handshake、target host key 與既有 alias 相符，client 明確顯示使用 publickey 完成認證，且遠端 `whoami` 與 `hostname` 均成功。這才是外網 SSH 已打通的有效證據。
- 使用者決定維持 public TCP 22。key-only 是核心安全控制；改用高埠只能減少掃描雜訊，不能取代 key-only。

# 維護入口

本機腳本以 `<windows-user-home>\configure-windows-sshd.ps1` 與 `<windows-user-home>\harden-windows-sshd-key-only.ps1` 表示；不得把真實使用者路徑寫入共享記憶。
