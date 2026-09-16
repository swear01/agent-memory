---
title: "改名部署須處理舊名稱的可載入產物"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 81a21abe361a8071ea74b50b2a9d7c1ab20c10ce2a958ec6d44d77de6f563ae4
---

# 改名部署須處理舊名稱的可載入產物

歷史唯讀 review 指出部署器只搜尋新名稱 jar，舊名稱 jar 仍留在 instance；新舊 mod ID 不同，不能依賴重複 ID 檢查阻止兩份同時載入。

改名部署要核對實際載入目錄中的新舊產物，測試兩者並存的情況，並依遷移契約處理舊產物。只檢查新檔名或編譯成功不足以驗收。

這是歷史靜態審查，未重跑 GameTest，也沒有修復完成證據。持久化 key 與 namespace 測試是另外的遷移問題；本筆不提供全面替換舊字串的規則。
