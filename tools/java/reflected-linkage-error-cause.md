---
title: "反射錯誤分類須查看被包裝的 LinkageError"
scope: tools/java
status: active
updated: 2026-09-16
evidence_digest: 2ddf307e27e81b14e0015ea9735c68923b0844dc5925aaca49b29b92b1ee56a5
---

# 反射錯誤分類須查看被包裝的 LinkageError

歷史 review 指出反射方法內的 LinkageError 被 InvocationTargetException 包住，落入一般 failed，而非預期 binary-incompatible。後續報告稱已按 cause 分類並通過 focused 檢查。

保留原始 exception cause，依真實失敗種類分類；僅搜尋 catch LinkageError 字樣不能證明反射入口會走正確分支。用真正經過反射包裝的例外驗證路徑。

後續報告明說 reflection 回歸是 source-level static test，沒有建立會拋錯的假模組；因此保留實際 runtime 驗證缺口，不宣稱動態路径已完整測過。
