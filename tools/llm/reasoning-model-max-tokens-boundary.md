---
title: Reasoning 模型在 OpenAI 相容介面下的 max_tokens 與空字串截斷
scope: tools/llm
status: active
updated: 2026-10-30
tags: [llm, reasoning, deepseek, token-budget, openai-compat]
---

# Reasoning 模型在 OpenAI 相容介面下的 max_tokens 與空字串截斷

## 現象與成因

當使用本地 Gateway（如 `127.0.0.1:35001/v1/chat/completions`）呼叫具備思考鏈（Reasoning）的模型（例如 `deepseek-flash`、`deepseek-pro` 或 DeepSeek V3/R1 相容架構）執行簡短文字任務（如分類代碼標籤、簡短主題命名）時：

- 若依慣例將 `max_tokens` 設為預期輸出內文長度（例如 `30` 或 `40`）：
  - 模型生成會在思考階段（Reasoning tokens）就迅速耗盡該額度。
  - 觸發 `finish_reason: "length"` 提前中斷。
  - 回應之 `choices[0].message.content` 返回空字串 `""`。
  - 對客戶端而言，看起來像是模型拒答或輸出丟失，而實際上是思考 token 佔用了全部的 `max_tokens` 預算。

## 處置準則

1. **預算分配下限**：
   - 在啟用思考鏈的模型呼叫中，即使最終預期內容僅有 2～5 個字（如類別名稱），客戶端 request 的 `max_tokens`（或 `max_completion_tokens`）至少應給予 `500` 到 `1000`。
2. **區分截斷與零候選**：
   - 收到空字串或格式解析失敗時，優先檢查 `finish_reason` 是否為 `"length"` 以及 `usage.completion_tokens_details.reasoning_tokens`。
   - 不應在未排查 token budget 的情況下直接判定為 prompt 理解失敗或模型能力不足。
3. **低延遲純文字提取**：
   - 若任務屬於高吞吐、確定性高的純文字提取或代碼標記，且不需要推理，應在 Gateway 設定或請求參數中關閉 reasoning，或選用非 reasoning 的 direct-completion 模型。
