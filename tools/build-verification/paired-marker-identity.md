---
title: "證據欄位必須成對綁定，不能只比獨立集合"
scope: tools/build-verification
status: active
updated: 2026-09-16
evidence_digest: a72a06f68beb185edc1f0f12b3844ccc61b2b6dd368203b3b6388f95a7da1817
---

# 證據欄位必須成對綁定，不能只比獨立集合

歷史第四輪審查回報，closure 分別比較 label 集合與 result-hash 集合，沒有把 label 綁到預期的 repetition、role、result 與 definition。審查者交換兩個有效 marker 的 repetition 後，closure 仍判 valid；一般測試通過也沒有抓到這個錯配。

完整性檢查應依獨立的預期清單逐筆比對完整關係，例如 (label, repetition, role, result, definition)，再驗證一對一、缺件、多件與 schema。各欄位各自相等，不能證明欄位之間的配對正確。負面測試應包含保留所有合法值、只交換關係的案例。

來源是歷史審查者自述的換位重現與測試結果，當輪未改檔；本次未執行該攻擊或驗證後續修復。同份報告中的程序回收與 symlink 問題是獨立機制，不能當成這個缺陷的根因。
