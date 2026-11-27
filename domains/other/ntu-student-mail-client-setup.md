---
title: "國立臺灣大學學生校園郵件（Webmail 2.0 / Roundcube）客戶端連線規範與校外連線除錯"
scope: "domains/other"
status: active
updated: 2026-11-28
---

# NTU 學生郵件：POP3/SMTPS 與驗收

- 學生 Webmail 用純學號當登入名稱，顯示郵件地址才含 `@ntu.edu.tw`。POP3 `msa.ntu.edu.tw:995`、SMTPS `smtps.ntu.edu.tw:465` 均 TLS 且 SMTP 需驗證。按使用者目前規則，寄學生信用 Himalaya `-a ntu`（只寄送），NTU M365 工作信走 Outlook connector，有工作信附件才用 Apple Mail。 [臺大計資中心郵件設定](https://jsc.cc.ntu.edu.tw/ntucc/email/mailsoftwaresetup.html)
- Exchange 的帳密驗證成功仍可能在送信回 `Client does not have permissions to send as this sender`，先查帳號類型和 envelope sender，學生照正式 SMTPS 設定。連線逾時與認證失敗分開看，核對網路、DNS、TLS 和埠號；不要由 timeout 就猜防暴力封鎖的時長或頻率。
- 手動設定 POP 客戶端時保持伺服器副本，讀回剛建立的確切帳號之留存設定，不以 `first pop account` 批次改既有信箱。先存草稿、附件核對和授權，送出後驗實際成功。轉收信箱可能延遲，不能把沒抓到新信當作沒寄出；必要時從獲准的學生 POP3 做 `STAT`／`TOP` 唯讀查，不能用 `DELE`。
- 郵件傳輸與 Himalaya 的已配置命令見 [[himalaya-ortie-setup]]；不要僅因 Apple Mail 有既存帳號就用它替代現在的寄送入口。

[早期 Apple Mail 配置與故障處理紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/domains/other/ntu-student-mail-client-setup.md)
