---
title: "群組維度正規化會改變既有特徵權重"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: 909024992926a5115c486fcbecbcb0249a3b7f2cec528a33bbd214acc82ec12e
---

# 群組維度正規化會改變既有特徵權重

歷史助手回報加入 1/sqrt(D) 群組正規化後，原本 bigram TF-IDF 對 cosine 的相對貢獻改變，指標退步；只恢復部分固定群組仍沒有恢復原行為。

若需求是保留舊行為並增加線性 scale，scale=1 必須真的是 no-op。將新的維度正規化視為另一次權重設計，對照原基線後再判定好壞，不能因公式整齊就當作等價重構。來源只有診斷與修改方向，沒有修正後準確率證據。
