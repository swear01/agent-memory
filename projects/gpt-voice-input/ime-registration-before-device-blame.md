---
title: "IME 不出現先核對 service 註冊路徑"
scope: projects/gpt-voice-input
status: active
updated: 2026-09-16
evidence_digest: 7044eb7b5aa208b30d61150dc826bb982a51aab36f551502f4fab4681f97ba96
---

# IME 不出現先核對 service 註冊路徑

助手回報 manifest 的 IME service 缺少 android.view.InputMethod intent-filter，先前重裝與 R8 調整未解；接著宣稱新版應出現，但使用者仍回覆不行。

先核對系統查詢是否能解析到預期 service，再測實際裝置的啟用與輸入流程。Manifest 或單元測試修正不等於使用者端問題全部解決，也不能先歸咎廠牌。

這是歷史診斷與未解的實機回覆，沒有真機修復證明。後續網路、key 與錯誤顯示要求是另外的功能工作。
