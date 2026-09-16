---
title: "未設定 service tier 不能在 bootstrap 時變成明確 Standard"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 8478b8b4e68c05554fadf3ed422bb3fc5c0b4fe9a92eee24dd18bc82442a80ca
---

# 未設定 service tier 不能在 bootstrap 時變成明確 Standard

歷史 bot 指出 currentServiceTier 的 undefined 被轉成 null，再由 keepalive 持久化，可能在使用者動作前覆蓋 Fast 或帳號預設；助手接著調查 resume 如何還原原值。

保留未設定、明確清除及具體值的三態語意，核對 bootstrap、setter、同步與持久化整條路徑。來源後续測試仍有失敗，助手發現 mock 缺 sessionInfo；不能把早先 CI 全綠或預定 guard 寫成修復完成。
