---
title: "去修飾規則不能把所有 dollar 前綴視為內部符號"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 0fd7920f555b991e37f2520783fb391ea065f1b4ec3ecc68739a15dacfac58c1
---

# 去修飾規則不能把所有 dollar 前綴視為內部符號

歷史助手指出排除樣式把以 $( 開始的函式脈絡變數也排掉，與應排除的 $string.const 混為一類。

以該輸出格式實際語法區分前綴，加入相鄰合法與應排除名稱的對照；pattern 命中數不代表抽出的變數語意正確。来源含修法建議與截斷的四十電路結果，沒有完整重跑證據，不能外推到所有 CBMC 命名版本。
