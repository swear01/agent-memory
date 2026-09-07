---
title: CPAchecker VGuide 實驗必須重用 Java transport
project: cpachecker
scope: projects/cpachecker
tags: [vguide, PredicateProposalClient, experiments, deepseek]
status: active
created: 2026-08-20
updated: 2026-09-07
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

## HTTP attempt evidence（PR211）

`PredicateProposalClient` 在既有 CPA log 寫入 `vguide-http-attempt-v1`：send 前
記錄 start，transport、HTTP status、parse 與 resource close 的共同邊界記錄 observed
terminal，另記一次 logical outcome。success 必須在 stream close 成功後才寫；
InterruptedException、error-body read failure 及 close failure 均不可遺漏或誤記成功。
Replay 不產生虛構 HTTP attempts；程序突然退出可能只留下 start，不能補成成功。
這是可觀察性修正，不是 scheduler 失敗重試的硬預算修正。

CPA LogManager 在分開的 message arguments 之間加空白，末尾也會附 source/level。
擷取時用 `line.split("VGuide LLM HTTP evidence: ", 1)[1].lstrip()`，再用
`json.JSONDecoder().raw_decode`；不能對整段尾字串 `json.loads`，也不能假設只有一個
空白。以 `(CPA log/run identity, logical_request_id)` 分組；request hash 加 ordinal
只在 client instance 內唯一。事件不得包含 payload、headers、憑證或 provider error body。

14 個 client tests 與 root 獨立重跑通過；最新 head 的 Gemini review5130800874
無待修意見。Merge `7159acae251afbac2e5f907bde01e41a8e0eb17c`。完整 native/Ant gate
並未全綠，不能將 scoped HTTP test PASS 擴張成完整 gate PASS。
