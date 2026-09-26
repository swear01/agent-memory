---
title: "國立臺灣大學學生校園郵件（Webmail 2.0 / Roundcube）客戶端連線規範與校外連線除錯"
scope: "domains/other"
status: active
updated: 2026-09-26
---

# 國立臺灣大學學生校園郵件（Webmail 2.0 / Roundcube）客戶端連線規範與校外連線除錯

## 背景與系統架構

1. **適用對象與系統本質**：
   - 2020 年 5 月以後入學之臺灣大學學生（學號如 `r14...`, `b13...`）。
   - 網頁介面入口為 `webmail2.ntu.edu.tw`，實質轉導至 `https://wmail1.cc.ntu.edu.tw/rc/`（基於 Roundcube 的 Linux 郵件系統）。
   - **非 Microsoft Exchange 架構**，且計資中心**不開放標準 IMAP**（Port 993 連線拒絕），收信僅支援 POP3。

## 客戶端（macOS Mail / 手機）設定參數

| 項目 | 正確設定值 | 常見錯誤與注意事項 |
| :--- | :--- | :--- |
| **電子郵件地址** | `<student-id>@ntu.edu.tw` | 需完整包含網域名稱 |
| **使用者名稱** | `<student-id>` | **必須為純學號**；嚴禁保留系統預設的「自動」，嚴禁附加 `@ntu.edu.tw` |
| **帳號類型** | `POP` / `POP3` | 嚴禁選擇 `IMAP` 或 `Exchange` |
| **收件伺服器** | `msa.ntu.edu.tw` | 埠號 `995`，必須勾選 SSL/TLS 加密 |
| **寄件伺服器** | `smtps.ntu.edu.tw` | 埠號 `465` (SSL/TLS)，必須勾選「外寄伺服器需要驗證」；**切勿改用 mail.ntu.edu.tw** |

## 寄件伺服器權限陷阱與校外連線超時除錯（核心教訓）

1. **切勿將寄件伺服器改為 `mail.ntu.edu.tw`（權限陷阱）**：
   - `mail.ntu.edu.tw` 是教職員使用的 Microsoft Exchange 伺服器。
   - 雖然其 Port 587 在校外看似可以通過 SMTP 帳密認證，但由於學生郵箱未建置於 Exchange 上，實際發送郵件時 Exchange 會嚴格拒絕代理，並拋出致命錯誤：
     `SMTP; Client does not have permissions to send as this sender`。
   - 因此學生發信**唯一合法的寄件伺服器只有 `smtps.ntu.edu.tw`**。

2. **校外 `smtps.ntu.edu.tw:465` 連線逾時（Timeout）真因**：
   - **官方政策無校外 IP 限制**：計資中心官方規範明確說明 `smtps.ntu.edu.tw` 支援校外寄信，其防護依賴「強制 SMTP 帳密驗證」而非純 IP 白名單。
   - **連線逾時成因**：當客戶端因舊密碼錯誤或 macOS 自動探測未加密 Port（110/25）連續失敗時，會觸發臺大伺服器端的防暴力安全機制（如 Fail2ban），將來源 IP 暫時靜默丟包（Drop / Timeout）15～30 分鐘，在介面上造成長達 60 秒的登入旋轉卡死。
   - **解決方案**：
     - 若遭遇 Timeout，請靜置等待 15～30 分鐘讓防火牆解除封鎖，或切換手機熱點/連線臺大 VPN。
     - 伺服器必須維持 `smtps.ntu.edu.tw`（Port 465 SSL），切勿病急亂投醫改換 Exchange 主機。

## 登入成功後必做安全配置

1. **避免伺服器端信件遭清空**：
   - POP3 協定預設會在本地客戶端抓取信件後將伺服器端郵件刪除。
   - 帳號新增完成後，必須進入 macOS 郵件「設定」→「帳號」→ 該校園帳號：
     - 將「收取郵件後移除伺服器上的備份」設為「**一個月之後**」或「**永不**」，確保網頁版 Webmail 2.0 信件完整保留。
