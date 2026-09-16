---
title: "BTOR2 寫入須符合解析器的最後換行契約"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 9ddcfbf147151e2fa2b4a334c88e6a32bb86d41d8d94620644681e6f66578e15
---

# BTOR2 寫入須符合解析器的最後換行契約

使用者貼出的 pono stderr 回報行尾預期 newline，stdout 是 error 與 b0；助手診斷缺少 trailing newline。

生成或追加 BTOR2 後檢查完整檔案語法與末行終止，再執行 solver。Parser failure 必須保留為輸入失敗，不能從相鄰 solver 摘要補出 verdict。

本筆只保存該解析診斷，未驗證修後結果；不採用相鄰自述的普遍加速、任意硬體適用或 soundness 結論。
