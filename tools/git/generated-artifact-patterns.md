---
title: "Ignore 樣式須用實際檔名驗證前後綴"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 08bde50569fa45a7bf51817ac41cc2b042c6fa32c69d3340166dff2bc2155fc2
---

# Ignore 樣式須用實際檔名驗證前後綴

歷史助手辨識出衍生轉換 netlist 應與正式輸出分開，接著發現 min_s953.v 沒有被既有後綴樣式排除，因為它使用 min_ 前綴。

提交前列出實際候選檔名，確認 ignore 或篩選樣式命中預期前綴與後綴，並先證實檔案確為衍生產物。不要只因名字像中間檔就排除正式來源；來源沒有最後提交或重跑結果。
