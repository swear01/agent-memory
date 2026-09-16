---
title: "失敗收集器須辨識各種正式 failure 格式"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: b6086a06ba383f4fdb03d7db1529d81650cb2f27bae0d787b9a743f5d62704b2
---

# 失敗收集器須辨識各種正式 failure 格式

助手承認 collect_failures 只匹配 Case-B 的 test FAILED 行，漏掉 Case-A trace mismatch。使用者後續輸出顯示五個 bucket 共十五個案例被收集，仍有三個 bucket 為零。

以實際來源日誌核對各類 failure 的解析與帳冊對應；保留 mismatch 和 assertion failure 的格式測試。零結果先檢查收集器是否漏讀，不能直接當 patch 無效，也不能直接判為測試映射錯誤。

後續 load/store 重映射成功只見助手自述，mtvec 沒有成功案例證據。缺少 patch-bucket 欄位是另一項 schema 問題，不併入這個根因。
