---
title: oracle 上 qmd vsearch 慢的原因是 1.7B query expansion，不是向量比對
scope: machines/oracle/qmd-vsearch-slow
project: agent-memory
status: active
confidence: high
evidence: 2026-10-22 在 oracle（Neoverse-N1 2 vCPU、11 GB、qmd 2.8.3、CPU-only llama.cpp）上分階段計時，並讀過 dist/store.js 的 vectorSearchQuery 和 dist/llm.js 的 expandQuery。
created: 2026-10-22
updated: 2026-10-23
tags:
  - qmd
  - oracle
  - performance
---

- `qmd vsearch` 不是純向量搜尋。它會先用 `qmd-query-expansion-1.7B-q4_k_m`（Qwen3 微調、grammar 限制、maxTokens 600）生成 lex/vec/hyde 擴展查詢，再把原始查詢加上每條 vec/hyde 擴展依序各做一次 embedding，最後才用 sqlite-vec 比對。
- 實測（load average 約 15–18，swap 已用 4 GB）。注意：這組數字是誤用「沒加 shared-memory env 前綴的 `qmd`」量的，所以 embedding 用的是預設的 embeddinggemma-300M。canonical 設定（`QMD_CONFIG_DIR`/`XDG_CACHE_HOME` 指到 `/var/tmp/qmd-memory-<host>`）的 embedding 是 Qwen3-Embedding-0.6B，比預設大一倍，會更慢。expansion 模型兩邊相同，都是 1.7B，所以結論不變：
  - embedding 冷啟動（runtime 加 300M 模型）25 秒，之後每次 0.95 秒。
  - expansion 一次 836 秒；模型已載入的第二次也超過 14 分鐘沒跑完。
  - 向量比對：1,498 個 chunk × 768 維，可以忽略。
- 擴展結果只在成功時寫進 `llm_cache`，所以超時被殺的查詢下次會整個重跑。
- 在 oracle 上的替代做法：
  - `qmd search`：BM25，不用 LLM，秒級。所有指令都要加 shared-memory skill 的 env 前綴；沒加前綴的 `qmd` 會打到空的預設索引並回報 "No collections"，這時不要在預設位置 `collection add`。
  - `qmd query $'vec: <查詢>' --no-rerank`：跳過擴展和重排，用預設 300M embedding 實測 46 秒，大半是冷啟動。
  - 需要完整的 `qmd query` / `vsearch` 時，到有 GPU 的 Mac 上跑。
- 中文查詢用純 vec 時排序不可靠：「Claude 遠端 session 能不能在 turn 中途插話」沒有把 hapi-claude-midturn-steer 排進前 3。關鍵字比較明確時，`qmd search` 反而比較準。
