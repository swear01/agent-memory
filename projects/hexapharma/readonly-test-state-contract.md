---
title: "測試狀態須保留正式型別的 readonly 契約"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 8d7511a725947c1e905610ce5dbd2f175e03d04fd698697e024a54436a8013d7
---

# 測試狀態須保留正式型別的 readonly 契約

使用者貼出的 TypeScript 錯誤顯示，balance 測試以手寫可變 sold 陣列接收 EconomyState，但正式型別使用 readonly SoldCount[]，因此賦值失敗。助手表示改用正式 EconomyState 型別。

測試的狀態宣告優先沿用正式匯出型別，避免手寫近似結構遺失 readonly 等契約。型別檢查失敗不能當成產品行為的 RED 證據；來源沒有後續 typecheck 通過紀錄。
