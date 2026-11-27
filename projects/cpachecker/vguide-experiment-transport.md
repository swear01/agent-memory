---
title: CPAchecker VGuide 實驗必須重用 Java transport
project: cpachecker
scope: projects/cpachecker
tags: [vguide, PredicateProposalClient, experiments, deepseek]
status: active
created: 2026-08-20
updated: 2026-09-23
---

- **同 prompt 用其他 HTTP client 出現 403 或得到另一種回答。** Production 路徑是 CPAchecker → `PredicateProposalClient`（Java HttpClient）→ router/provider。探針可加極薄 Java CLI／JShell；完整 context、validation、injection 用 CPAchecker replay。Python urllib probe 的初次 403 來自 upstream，換 user agent 或 production Java 才 200；它不能用來判斷 prompt 品質。
- **HTTP 200 或 SSE DONE 被當成完整 JSON。** 分開保存 transport status、DONE/EOF/error、nullable finish reason/usage、content UTF-8 bytes/hash 和 parse reason。Stream close 失敗時仍保留先前 parse 證據，記 `stream_close_failure`，不提前記 success。`recordsStreamCloseFailureBeforeSuccess` fixture 覆蓋 second close。重播保留缺的終局欄位為 unknown；沒有 finish_reason 不歸因 token cap。Meta wire 固定 `loop-head-candidate-v1` response_format 時，prompt 自由文字仍受 schema 限制，先核實 request。
- **HTTP events 只剩 start 或計數被重複累加。** `vguide-http-attempt-v1` 對每次 attempt 記 start 和 observed terminal，每組 logical request 另有一個 outcome；process 突停的 start 保留未知。用 `line.split("VGuide LLM HTTP evidence: ", 1)[1].lstrip()` 取事件，再用 `json.JSONDecoder().raw_decode` 去除 LogManager 後綴。按 log/run id 與 logical request id 分組，驗連續實際 attempt indices；`max_attempts` 是 cap。各階段累計依唯一 slot，勿把已包含 first pair 的總帳再相加。reasoning_tokens 是 completion 的細分；未回 usage 不記零。
- **Round 裡的 response hash 對不上整份 cache JSON。** 比對同 task/request/ordinal 的 `llm_rounds.jsonl` 和 cache 裡 `result.content()` 的 UTF-8 hash。外層 JSON 檔案 hash 另計。telemetry 不落 payload、憑證、headers 或原始 provider error body。細節見 [實驗 provenance](vguided-experiment-provenance.md)，整體結果與 transport 驗證分開。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/cpachecker/vguide-experiment-transport.md)
