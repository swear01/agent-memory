---
title: "非空 summary 檔不能直接充當可續跑完成標記"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: b2ee7c62edee0124a3794753984e8d44cbff01cf977ada8a4716d7fef889fc47
---

# 非空 summary 檔不能直接充當可續跑完成標記

歷史 review 指出 runner 直接寫 summary，若寫入途中崩潰，留下的非空檔可能使 resume 誤判已完成；當次整體 review 仍為 FAIL。

將完整 summary 驗證後原子發布，恢復時檢查其必要欄位與完成契約，並用中途失敗測試驗證。存在 XML、已有 artifact manifest 或一般測試通過，都不能代替完成證據。來源另有 teardown 及 provenance 覆写問題，屬不同機制且沒有在此證實修好。
