---
title: "外部工具輸出解碼失敗須保留原始 bytes"
scope: tools/python
status: active
updated: 2026-09-16
evidence_digest: ff15d493fb26309d918b413285c48463e418d6f6b52e5fc109d6f84c389c651e
---

# 外部工具輸出解碼失敗須保留原始 bytes

使用者 traceback 顯示 subprocess 在解碼 stderr 時遇到非 UTF-8 byte 而退出；相鄰另有 BTOR2 parser error，不能先假定兩者同因。

依工具輸出契約決定 encoding；對不可靠的診斷串流保留原始 bytes，再另外產生可讀顯示。不要靜默丟掉無法解碼部分，也不能在尚未取得退出狀態時補出 solver verdict。

來源沒有編碼修復結果，不採用手寫 BTOR2 格式已修好或 memory leak 都同因的結論。
