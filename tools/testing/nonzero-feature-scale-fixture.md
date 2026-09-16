---
title: "測特徵 scale 效果須讓該組特徵非零"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 9a2b6a258c33b4801958371d25e88903de235b70c9dba3b74766f8e0c6d8bc9d
---

# 測特徵 scale 效果須讓該組特徵非零

原始斷言調高 mnemonic_histogram scale 後期待總範數變大，實際相等；助手指出該 fixture 的 histogram 全為零。

先確保被調整的特徵組在測試中有非零值，再檢查對應 block 的變化與其他 block 不變。零向量乘不同 scale 仍是零，不能由此推論 scale 被正規化消掉。

來源沒有修後重跑。除以 sqrt(D) 不會普遍消除 scale 的作用；不採用草稿的數學錯誤或未證實的最大範數宣稱。
