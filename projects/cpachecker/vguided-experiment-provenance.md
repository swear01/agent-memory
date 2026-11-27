---
title: CPAchecker VGuide 實驗 provenance 與 preregistration 陷阱
project: swear01/cpachecker
scope: project
tags: [vguide, experiments, provenance, preregistration]
status: active
created: 2026-08-30
updated: 2026-09-30
---

## 凍結與啟動

- **正式實驗的 comment 看得到，發出時間卻在 job 之後。** 先以 `gh issue comment ISSUE --repo OWNER/REPO --body-file COMMENT.md` 發布，讀回遠端本文、hash、時間，再 launch。避免 shell 插值 backticks／`$()`；#166 G0 的 comment 晚一秒使正式證據無效，事後重跑也不能變回 blind preregistration。
- **同名 benchmark 或新 checkout 看似等價。** 凍結有序 YAML、原始 `input_files`、實際 source bytes/hash、property、資料模型及 runtime。預處理 `.i` 不可換成同名 `.c`；ILP32／LP64 的 exact argv 要分別呈現 Linux32／Linux64。hard218 由 224 依序排除 6 題所得，歷史 764 和 PR99 未找到原始 bytes 的 hash 都不能替代。
- **Manifest 的 HEAD 和實際執行檔不同。** `VGuideAnalysisDumper.readGitCommit()` 只在 CWD 讀 `git rev-parse HEAD`。每 arm 綁該 worktree 的 CWD、launcher、startup version、classes/lib 檔案證據，不回寫舊 dump。詳見 [build 與 runtime 驗證](build-verification.md)。
- **Runner 回報啟動但沒有出現題目。** CPU 隔離使用 frozen label→SSH allowlist，claim 後核實 hostname；禁止透過本機 fallback 變出額外主機。#296 runner 用 `mkdir(exist_ok=False)`，預建 output parent 而非正式 child。讀實際 preflight 和第一筆 dispatch。正式 timing 要保留運行中競爭檢查；前後空閒及其他 process affinity 只能輔助診斷，不能自行證明全程 core residency 或各題受影響。

## Replay 與 intervention

- **Exact replay miss／序列不同。** Cache key 是 production Java 整個 request JSON bytes 的 SHA-256（含 provider/model/schema/options/stream/reasoning 等）。歷史 DeepSeek replay 設 `VGUIDE_LLM_PROVIDER=deepseek` 與 `VGUIDE_LLM_MODEL`，retired `DEEPSEEK_MODEL` 沒作用。以原始 `llm_rounds.jsonl` 重建時序，候選注入後的下輪 hashes 可以不同。保留 expected/actual hash、log 和失敗分類，找原因開新 attempt；不改寬 key 或改舊 raw。#296 的 missing second response 是 REPLAY_MISS，CPU watchdog 是 REPLAY_BUDGET，fresh verdict 各自解讀。
- **FIRST_SPURIOUS ablation 看不到 history 差異。** 首輪 request 往往先建 context、之後才記錄本輪 CE/outcome。先驗每份 raw request 真有變因，保持其他輸入相同。完成回合、logical request、HTTP attempt 分開；`37064e4` 的 SAFE repair 使一個 scheduled round 在有兩次 retry 時可達 2 logical／6 HTTP。實際事件核帳，不能以 HTTP 200、SSE `[DONE]`、`llm_calls=0` 或無 cache 代替 response JSON 完成和 transport 計數。
- **Raw candidate ordinal 被當 production predicate_id。** 凍結 response 與每筆 head、canonical formula、source_profile 對照。Replay-only 設 `VGUIDE_LLM_REPLAY_DIR`，用 `vguide.replayInjectionMode` 的 FULL／SUPPRESS_ALL／EXCLUDE；EXCLUDE 檔按 `head=N<number>;formula=<canonical SMT from dump>;provenance=<source_profile>` 寫且對照跨輪累計，保留 outcome 與缺匹配證據。Placement 數可多於 unique candidates。

## 終局、成本與報告

- **非空 verdict 或 dump 數被算成解題。** 用官方 label 分類 correct／wrong／unresolved，保留完整分母，UNKNOWN 非 correct 也非 wrong。Refinement attempts 含可行反例及失敗，completed spurious dump 列可少一筆；用原始 log 診斷，保留 partial coverage。分類 native crash、Java heap OOM、cgroup OOM、timeout、wrong 與無終局的 infra；#296 舊 fullset 把 `exit=1` 的 Java heap OOM 誤列 UNKNOWN，應一致依 pilot 列 INVALID、保留成本和原紀錄。
- **某台 final accounting 用了全域數字。** 依 `row.host` 篩選，再分 launched、terminal、預期題數。完整成本看所有實際 attempts 與服務及 cleanup，未知 usage 保持未知。只將 matched pairs 和實際 complete-response rows 做各自分母，不能用全資料的成功數除以局部集合。
- **STOP sentinel 已出現但 coordinator 沒收到結果。** 等真實 supervisor 寫完 summary，再送 hash、計數、exit 與 STOP reason，核驗送出回條。CLI accepted 不等於已讀。保存 frozen runtime 與當前 PR head 的區別；review 通過、merge、分派結案和實驗受採納分開。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/cpachecker/vguided-experiment-provenance.md)

- 啟動前核對實際選集用途、成員順序及內容，避免把 `excluded_trivial_solved` 當 non-trivial 目標。誤跑結果保留其真實範圍，重跑用獨立輸出目錄。

[合併來源](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/selected-dataset-identity.md)
