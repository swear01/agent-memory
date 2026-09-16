---
title: "套歷史 fix 前核對 pin 真正存在的模組與訊號"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: 594983fc93c223526b3205aa086f1903677c911aa006559b8f094c6da2ed4f1c
---

# 套歷史 fix 前核對 pin 真正存在的模組與訊號

歷史助手發現 batch 把 stepie 修正列在只有註解變更的 controller，實際程式改動位於 CSR 模組；另一個 fix 引用的效能計數訊號在目標 pin 不存在。

讀完整 diff 與指定 revision 的實際 tree，分開確認語意修改、註解及所需訊號。不能憑現代路徑或相似模組名假設舊版本可套用。來源只到調查，沒有移植、編譯或因果測試結果。
