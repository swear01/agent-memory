---
title: VGuide JSONL streaming 與 Writer ownership
scope: projects/cpachecker
status: active
updated: 2026-09-10
---

大型 refinement dump 若先呼叫 `ObjectMapper.writeValueAsString(row)` 再寫檔，會在
既有 JSON tree 之外配置完整 serialized String；#103 ECA replay 的 heap OOM stack
落在這條路徑。這只確認失敗配置位置，不能證明它是唯一記憶體來源。

PR #240 改成 UTF-8 append BufferedWriter，直接串流 JSON。若呼叫端還要寫 JSONL
newline，需讓 Jackson 保留外層 Writer ownership：

```java
JSON.writer().without(JsonGenerator.Feature.AUTO_CLOSE_TARGET).writeValue(writer, row);
writer.write('\n');
```

外層 try-with-resources 負責 close；停用 generator 關閉 target 並不表示不關檔案。
以 multiple rows、Unicode/escaping、大內容、close/reopen 與 append failure 驗證
既有完整行仍保留。此改動移除完整 serialized String，沒有移除原 JSON tree 或
保證任意大資料不會 OOM。Merge `d195e94cc51f5f14f620d3a1c0c6cb906a12eb2f`；
source `src/org/sosy_lab/cpachecker/cpa/predicate/vguide/VGuideAnalysisDumper.java`，
focused `VGuideAnalysisDumperTest` 5 tests 通過；實際 ECA replay 效果另行驗證。


PR #249 補上 row-local block formula 字串快取，merge
`00b891c3aabc82cb5f84a5fbb00c8690088f50dc`。#241 保留的 ECA refinement
單行仍有 663,014,820 bytes；其中一個 47,971,285-byte JSON 字串值在
`block_formulas[1].smt` 與十二個 `validated_predicates[*].block_formula_smt`
共出現十三次。串流 JSON 不會消除 tree 內先建立的這些重複 String。

`validatedPredicatesJson` 現在用函式內 `IdentityHashMap<BooleanFormula,String>`，
同一 block formula 物件每 row 只 stringify 一次；不同物件不假設語意等價，
下一 row 重新計算。輸出的欄位、順序、字串值與檔案大小不因這個快取縮減；
不能把它宣稱為 OOM 已治癒。六個 focused dumper tests 通過，其中回歸測試
驗證同物件重用、不同 block 保留、跨 row 不重用及 JSON 值相同。

再次診斷巨大 dump 時，先按 JSON 欄位與內容 hash 分清輸出重複和記憶體配置；
避免用一般 JSON load 或切出完整大字串來做「低記憶體」分析。這次 report-local
stdlib profiler 使用 mmap 掃描與固定大小 memoryview hash chunks。證據：
`<experiments-root>/reports/issue248-dump-20260910/REPORT.md`；原始 refinement
SHA-256 `61e9001439dfc241a1cd940fd1b3d07277a423c44ec1d6dd9a072f5180e13854`。
