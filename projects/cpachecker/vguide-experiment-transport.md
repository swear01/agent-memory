---
title: CPAchecker VGuide 實驗必須重用 Java transport
project: cpachecker
scope: projects/cpachecker
tags: [vguide, PredicateProposalClient, experiments, deepseek]
status: active
created: 2026-08-20
updated: 2026-09-08
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

## Attempt cap 與分階段計數

`max_attempts` 是上限，不是實際次數；第 1/3 或 2/3 次成功都有效。每個 logical
request 的 observed attempt index 必須連續，且每次各有 start 與 terminal，再有一次
logical outcome。只比對最後一次 terminal 會漏掉中間缺失；不能要求一定耗盡 cap。

分階段續跑時，以唯一 slot ID 的已保存紀錄建立同一份 cumulative ledger。
若完整 slot-result 清單已包含 first pair，就不能再加 first-pair 的摘要數字。
Issue #218 的 slot15 曾因此被 checker 誤判超額；保留失敗 receipt，重新核對原始
execution 與 HTTP events 後只修正衍生計數，沒有重跑或替換有效結果。必須區分
checker accounting failure 和 provider/solver failure，避免為修正統計而再付費執行。

Provider usage 的 `completion_tokens_details.reasoning_tokens` 是 `completion_tokens`
內的細分，不能再加到 completion 或 total。失敗 attempt 未回傳 usage 時保留 unknown，
不可補成零或從其他 attempt 推估。

## SSE 終止證據與內容解析分開（PR228）

HTTP 200、SSE `[DONE]` 與 candidate JSON 可解析是三個不同結果。已收到 DONE 的
content 仍可能是不完整 JSON；缺少 observed `finish_reason` 時，不可把截斷直接歸因於
token cap。保留 HTTP status、DONE/EOF/error、nullable finish reason/usage、UTF-8
content byte length/SHA-256 與 parser reason 即可，不需增加原始 headers 或完整 wire log。
Replay cache 沒有保存的 terminal evidence 維持 unknown，不從成功 replay 推造 HTTP 事件。

`PredicateProposalClient.proposeWithUsage()` 的外層 try-with-resources 可能在 parser 已成功
後才因 `bodyStream.close()` 丟出普通 IOException。只從 `StreamParseException` 取 evidence
會漏掉已知 DONE、content hash/length、finish reason 與 usage；應保留成功解析得到的 evidence
作為 close-failure fallback，仍記 `stream_close_failure`，不可提前記 success。
既有 `recordsStreamCloseFailureBeforeSuccess` 的 second-close fixture 可直接驗證此邊界，
不必增加另一套 HTTP 模擬。PR228 的 client/dumper 21 個 focused tests 通過；這不是 live
provider 或完整 verifier 的驗證。
