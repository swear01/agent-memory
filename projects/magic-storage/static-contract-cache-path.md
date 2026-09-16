---
title: "靜態 family 建立 contract 後要核對重建時會命中 cache"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 41c0e772eff20f2528d0d1dcd3273263a32a7dcee566f8a9a3f10e7e9dd9ee7b
---

# 靜態 family 建立 contract 後要核對重建時會命中 cache

歷史 review 指出 single-plan typed family 直接建立 contract，沒有填入 typedContractCache，後续 Craftable rebuild 可能每次重新建立。

依靜態與動態 family 的契約檢查 cache 寫入、讀取與失效路徑，讓測試觀察重複 rebuild 的實際工作。來源只是 review finding，沒有量測退化或修復通過。同行的舊測試數量、pruning fixture 與 ancestry validation 是獨立問題，仍留在來源帳本。
