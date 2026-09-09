---
title: CPAchecker verdict audits
scope: project
status: active
updated: 2026-09-09
---

# CPAchecker verdict audits

- For SV-COMP verdict triage, the frozen dataset/official SV-COMP label remains the correctness ground truth. Agreement from k-induction or independent tools is diagnostic evidence and must not relabel a disagreement.
- Keep arithmetic/bitvector, array-permutation, and heap/list-structural explanations separate unless source inspection identifies a shared cause. A machine-model mismatch is only a hypothesis until independently reproduced with exact provenance.
- #54/#56 的本地 source/task/property hashes 與 frozen records 比對，和官方結果表的 task/property 對照是兩種證據。重用已驗收的歷史本地來源紀錄，不能直接標成官方 source bytes/data model 完全一致；必須另有官方來源與模型的可核對證據，否則明記 name/property-only。歷史十二個 disputes、六個 crash/unqualified tasks 和 #178 LP64 observation 保持分開。
- BenchExec HTML 表內有結構化 `const data` JSON；用標準 JSON parser 讀取，從實際 tool metadata 找 status 欄位，驗證 task 唯一性和工具/欄位數量。不要用 HTML regex 配硬編碼工具清單的位置來推定結果歸屬。
- 外部結果分類要區分 mixed、agreement-with-expected、disagreement-with-expected 和 no-definite-result。非空且一致反對 expected 的集合不是「沒有明確結果」。#56 的 `sll-01-1` 曾因這個 else 分支誤報；修正分類仍不改 frozen expected verdict 或 official-wrong 計數。
- 驗證依據：#56 current verdict cross-check 的 raw public table 與 root review，`<experiments-root>/reports/issue56-current-verdict-crosscheck-20260908/`。

- Counterexample 輸出能力要追完整呼叫鏈，不能只讀 ARG witness options。已驗證 runtime208 的 `PredicateCPARefiner` real-error branch 呼叫 `PathChecker`，後者可透過 `counterexample.export.formula` 與 `counterexample.export.model` 加入 SMT formula / assignment，最後由 `CEXExporter` 內迭代 `counterexample.getAllFurtherInformation()` 的迴圈寫出。原 run 使用 `--no-output-files` 而缺檔，不能推論 exporter 不支援。精確路徑與模型取得成功才有相關輸出；timeout 或 imprecise path 的缺漏要照實記錄。
- `PathChecker.createCounterexample` 的模型重建是同一 CPA 分析內的既有 SMT 操作；和另行啟用 `analysis.checkCounterexamples` 或啟動第二個 verifier 不同。做 allocator 診斷時兩 arm 必須保持相同的輸出／檢查設定，只改預定的 allocation option。依據：#236 root source review，`<experiments-root>/reports/issue236-allocation-diagnostic-20260909/root-source-export-review.json`；不代表已執行或證實任何 wrong-verdict 根因。

- 讀取 allocator counterexample model 時，不能把 `malloc@k` 直接當成回傳地址。已核對 `DynamicMemoryHandler` 的失敗配置：fresh pointer-typed nondeterministic value 只決定 `ite(value != NULL, successful_alloc_address, NULL)`；必須再追實際程式指標的 SSA 等式及 source dereference 順序。模型中的非零 selector 也不等於分配到的地址。
- 若編碼反例先對 NULL 解引用／寫入，再觸發 assertion，這能證明該分析模型走過 nullable-allocation 分支，不能單憑它宣稱存在 defined-C assertion counterexample、推定官方 benchmark 假設 allocation 必成功，或更改 frozen label。`memoryAllocationsAlwaysSucceed=true` 後變成 UNKNOWN 也不是修復成功。依據：#236 source／SMT／model 對照與 root acceptance，`<experiments-root>/reports/issue236-counterexample-interpretation-20260909/`。

- Counterexample GraphML 預設可能是 gzip：`CEXExporter.compressWitness=true`，`writeErrorPathFile` 會在設定的 `.graphml` 後附加 `.gz`。只 glob `Counterexample.*.graphml` 會把有效 witness 誤報為缺失。#236 三個 FALSE arms 的既有 `.graphml.gz` 已逐檔雜湊、解壓並確認 GraphML XML root；UNKNOWN arms 無 witness。收割時應同時盤點 plain/compressed 檔案，保留原始壓縮位元組與 hash；凍結後才發現漏盤點時加補充 receipt，不改 raw/pattern-specific inventory，也不把 XML 可讀誤稱獨立 witness validation。依據：`<experiments-root>/reports/issue236-allocation-diagnostic-20260909/root-compressed-graphml-correction.json`。
