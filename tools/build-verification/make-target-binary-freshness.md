---
title: "Make 沒重建時先核對真正 binary target"
scope: tools/build-verification
status: active
updated: 2026-09-16
evidence_digest: 74dad404cdd7ad168a36f323bd7cda2f3985c124a1c878898c7230e7d2af44ae
---

# Make 沒重建時先核對真正 binary target

助手發現 make pono 沒更新舊 binary，改查實際 target pono-bin；使用者隨後貼出新時間戳與 initial-predicates help 選項。

核對 build graph 的實際 target 與執行產物，使用新功能驗證確實載入新 build。時間戳只是線索，不能以命令沒有報錯或舊可執行檔存在判建置完成。

後續仍見候選 build 失敗與零注入，另次 UNSAT 又沒有 INVAR certificate；新選項可用不代表 predicate 或原模型證明已驗證。
