---
title: "兩份缺失證據相等不代表完整性"
scope: "tools/build-verification"
status: active
updated: 2026-09-15
evidence_digest: 55d99648429e63674edb61e79c8ef2672e43af34a34452232f4c5d4ccff90eef
---

# 兩份缺失證據相等不代表完整性

歷史唯讀分析回報：candidate 的 hierarchy 與 top-level structural_hierarchy 同時移除後，驗證器仍因 None == None 通過，並退回名稱線索；既有測試只刪掉其中一份，沒有覆蓋同時缺失。來源明示沒有修改檔案。

比較兩份證據之前，先驗證各自是否存在、符合 schema，並對照獨立的預期 inventory 檢查完整性。負面測試應包含兩份一起缺失，以及只缺一份；不能只檢查多餘項目而漏掉遺失項目。

來源提出綁定 artifact、classpath 與完整分類集合的 digest 作為修正建議，並非已完成修復。自包含 digest 無法抵抗內容與 digest 一起重算；需要更強防竄改保證時，須有獨立可信依據或從 exact artifact 重建。這次查核支持歷史分析的陳述範圍，未重新執行該 mutation。
