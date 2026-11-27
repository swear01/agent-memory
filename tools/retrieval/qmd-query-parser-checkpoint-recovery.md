---
title: "QMD 查詢語法失敗要保留已通過的回讀進度"
scope: tools/retrieval
status: active
updated: 2026-09-16
---

# QMD 查詢語法失敗要保留已通過的回讀進度

本次 Issue25 回讀有兩種查詢端失敗：vec 文字中的 ant -version 被已安裝 QMD 的 validateSemanticQuery 當成不支援的負向條件；另一查詢的 Major／Blocker 經 sanitizeFTS5Term 移除全形斜線後合成 MajorBlocker，與索引分詞不符。這些是本次版本與輸入的實測結果，不代表所有 QMD 版本都有相同行為。

先分開檢查查詢剖析、詞彙匹配與文件內容。把命令列旗標作為字面內容加上 code delimiters；詞彙查詢中的全形斜線改成空格。只調整 query，不改筆記來迎合查詢，也不放寬 top10、精確文件身份或全文比對。

兩次恢復均保存原始錯誤輸出與雜湊，核對已完成前綴的筆記身份、內容雜湊及回讀結果後，只續跑未完成部分。當時以相同驗證條件完成 lexical、hybrid 和逐行回讀；沒有重做已成功部分，也沒重新 embed。查詢成功只證明可檢索，來源語意另外查核。[歷史回讀前綴](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/retrieval/qmd-query-parser-checkpoint-recovery.md)
