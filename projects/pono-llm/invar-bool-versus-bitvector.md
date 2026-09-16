---
title: "讀取 INVAR 時不能把 Bool 再當 BV1 比較"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 391b13020d938f6c76d7dcb3eb67b0a1d53c2e1c543b91e33a2c19960f7d7ef7
---

# 讀取 INVAR 時不能把 Bool 再當 BV1 比較

原始 Z3 traceback 顯示 parse_invar 把外層 Bool 的 INVAR 再包成與 #b1 的比較，兩個案例都因 Bool／BitVec1 型別不相容而失敗；歷史助手承認 parser 多包了一層。

依實際 SMT sort 處理公式，不以內部含有 bitvector 就推論整個 expression 是 BV1。來源後來回報 C2 拒絕人造假陽性與 gcd，尚準備跑真有效 invariant 的正例；parser 可讀及負例遭拒不等於 checker 完整合格，也不證明原電路不安全。
