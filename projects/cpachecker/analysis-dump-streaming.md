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
