---
title: DVLab IKEv2 VPN 雙軌架構與 Windows 用戶端排查
scope: projects/dvlab-mis
project: dvlab-mis
status: active
updated: 2026-09-17
tags: [vpn, ikev2, windows, powershell, eap, psk, dvlab-mis]
---

# DVLab IKEv2 VPN 雙軌架構與 Windows 用戶端排查

## 架構現況

- 實驗室 IKEv2 VPN 採平台雙軌制：
  - Apple / Android：端點走純 PSK（Pre-Shared Key）模式，不需個人或共用使用者帳號密碼。
  - Windows 10 / 11：現行版本為 `DVLab IKEv2 EAP`，走獨立端點與 EAP-MSCHAPv2 認證，需匯入專用 CA 根憑證並輸入共用認證帳密。
- 正式安裝包位於伺服器 `<remote-home>/dvlab-vpn/latest/DVLab-IKEv2-VPN.zip`，由發布流程維護。排查連線問題時，應先確認用戶端取得的是最新釋出包，不可單憑本機暫存目錄的歷史 ZIP 作為依據。

## Windows 腳本 PropertyNotFoundStrict 根因與處置

- **現象**：同學執行 `Install-DVLab-IKEv2.ps1` 時中斷，報錯 `Set-RasphonePreSharedKey : 在此物件上找不到屬性 'Count'。請確認該屬性存在。`（`PropertyNotFoundStrict`）。
- **根因**：
  1. 該同學使用的是 2026-07-17 的早期安裝包。早期腳本在 Windows 上嘗試寫入 PSK，函式 `Set-RasphonePreSharedKey` 包含 `$lines = Get-Content -LiteralPath $PbkPath -Encoding Unicode`。
  2. Windows 原生產生的 `rasphone.pbk` 常為 ANSI 或 UTF-8。強制以 Unicode 讀取會使換行無法識別，回傳單一 scalar 字串（或空白檔案為 null）。
  3. 腳本開頭設有 `Set-StrictMode -Version Latest`，存取 scalar 物件的 `.Count` 會直接觸發例外終止。
- **處置**：
  - 引導同學直接自伺服器重新取得最新正式安裝包（已切換至 EAP 模式，不再呼叫 `Set-RasphonePreSharedKey`）。
  - 若需維護舊版或撰寫類似 PowerShell 腳本，檔案讀取行數應一律使用強制陣列 `@(Get-Content -LiteralPath $PbkPath)`，並動態偵測檔案 BOM / 編碼；修改電話簿等次要設定需加 `try-catch` 防禦，避免阻斷主體 VPN profile 建立。

## 文件與公開說明邊界

- 公開說明文件保留給成員閱讀之安裝指引，不直接列出明文認證帳密，引導成員透過驗證帳號由內部伺服器取得包含設定與認證資訊的安裝包。
- 排查同學詢問「是否有帳號密碼」時，需先區分作業系統：Apple/Android 無帳密，Windows 需填入安裝包隨附之共用認證。
