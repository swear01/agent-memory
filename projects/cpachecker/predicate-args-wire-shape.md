---
title: "Predicate args 形狀不符不能靜默變成沒有候選"
scope: projects/cpachecker
status: active
updated: 2026-09-16
evidence_digest: 35b7bb727d9867f6fe368401d16bc19fce4d8ac15b704b65731dd588cb524ba0
---

# Predicate args 形狀不符不能靜默變成沒有候選

歷史助手回報 mock sidecar 使用字串 args，但 parse_predicate_args 只掃描物件節點，因而忽略字串；它接著提出改用 nested ref nodes 並檢查正式 LLM 的格式。

以實際 wire schema 驗證產生端與解析端，將不支援的參數形狀明確回報，避免無聲漏掉候選。來源的注入加速是 mock 相關自述，正式 DeepSeek 全測仍是下一步；不能記成正式模型已達成的效能。
