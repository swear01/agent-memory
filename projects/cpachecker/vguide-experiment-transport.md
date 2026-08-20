---
title: CPAchecker VGuide 實驗必須重用 Java transport
project: cpachecker
tags: [vguide, PredicateProposalClient, experiments, deepseek]
status: active
created: 2026-08-20
updated: 2026-08-20
---

# VGuide 實驗 transport 邊界

VGuide production 是單一 Java 路徑：CPAchecker → `PredicateProposalClient`（Java
`HttpClient`）→ OpenAI-compatible endpoint/router → provider/model。

Prompt probe、A/B 與 replay 實驗必須直接重用 `PredicateProposalClient`，可加極薄的
Java CLI/JShell 入口；需要完整 context/validation/injection 時則直接跑 CPAchecker
replay。不要另寫 Python/curl HTTP client，因為 headers、request construction、retry、
response cache 與 error handling 不同，結果不能代表 production。

已驗證案例：Issue #124 的臨時 Python `urllib` probe 對
`http://127.0.0.1:35001/v1/chat/completions` 收到上游 `api.commandcode.ai` Cloudflare
HTTP 403；同一 Python request 只把 User-Agent 改成 curl 即回 200，而正式 Java
`PredicateProposalClient` 單獨呼叫也回 200（`{"ok": true}`）。因此該 Python probe
兩次嘗試均無效且沒有模型 inference，不能拿來判斷 prompt 品質或 provider 狀態。
