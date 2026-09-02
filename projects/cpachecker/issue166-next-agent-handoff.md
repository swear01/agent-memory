---
title: CPAchecker issue 166 next-agent handoff
project: swear01/cpachecker
tags: [vguide, issue166, handoff, replay, schema11]
status: superseded
created: 2026-09-02
updated: 2026-09-02
---

# 狀態

> 2026-09-02：此接手狀態已由 schema-11 deterministic controls 正式通過而 supersede。
> 後續 Python classifier 環境與目前狀態見 `issue166-python-environment.md`；以下保留為當時的
> recovery provenance，不再當作 active next step。

#166 仍 open，acceptance 未滿；目前沒有 run。PR #169 的 schema-11 instrumentation 已 merge，
最後確認的 milestone commit 是 `4a14d72cc15dd38263a2430f4064b352a9b2782c`，但接手者必須先
fetch 並以當時最新 `origin/main` 建立新 claim/worktree。禁止修改主 checkout，也不要沿用或清理
其他歷史 issue166 worktrees。

2026-09-01 的 frozen control attempt 在第一個 response 前 cache miss；empty 無有效 consumer
evidence，complete 未啟動，external call=0。Frozen protocol/harness/stop artifacts 必須保留，不能
原地補 cache 重跑。

# 下一步唯一主線

1. 先讀 issue #166 全部 comments、experiment protocol、GitHub Wiki、#139/#147/#148/#149 與
   `<remote-home>/cpachecker-experiments/runs/issue166_semantic_precision_census_20260901/NEXT_AGENT_HANDOFF.md`。
2. 在新 attempt 目錄，透過 production Java client + local deterministic oracle +
   `VGUIDE_LLM_RECORD_DIR`，分別建立 empty/complete 的 current Meta/Muse cache。明確固定
   provider=`meta`、model=`muse-spark-1.2-contributor`、thinking=`disabled`→minimal、CPA token
   option=1024；禁止 `DEEPSEEK_MODEL`、舊 hash 與跨 arm 共用 sequence。
3. 每個 ordinal 驗證 oracle raw-request SHA = Java recorded request hash、response SHA、prompt
   SHA、schema/options/namespace/order；再以純 `VGUIDE_LLM_REPLAY_DIR` 跑第二次 qualification。
4. Preflight 失敗要自行保存、修 harness、重跑；不是 task stop。兩臂 qualification 通過後，才
   把 per-arm prompt/request/response/cache hashes 與 exact command 用 `--body-file` preregister
   到 #166，讀回遠端確認後再 launch。
5. Valkyrie 當下有其他 workload，不在這台啟動長 CPU/GPU job；等 idle-ready 或依 protocol 用
   Athena/Cthulhu。Formal retry 要 5x1 秒 load check、P-core taskset、HAPI job、pure replay、零
   external model call。
6. Acceptance：empty 無注入且 `UNKNOWN`；complete 不修 response、不挑 atom/head，完整五個
   candidates 依原宣告 2/3 heads 注入且 `TRUE`；兩臂 schema 11 formula/state 對齊且 artifacts
   完整。Generation contract 與 consumer verdict 分開回報，未滿不要 close #166。

# Stop semantics

`fail-closed` 只做 `attempt_stop`。Agent 隨後必須做 `recovery_action` 並建立合規新 attempt；只有
缺 authority/credential、需使用者改變研究問題的決策，或同一不可控 blocker 經至少三次有記錄
的 recovery 仍重現，才是 `task_stop`。不能 silent fallback，也不能因 attempt stop 停止研究工作。

# 已知根因與不可重做事項

Exact-request SHA 設計正確；舊 harness 使用被 Java 忽略的 retired `DEEPSEEK_MODEL`，實際 default
成 Meta/Muse，並錯誤重用 #149 DeepSeek hashes。Prompt/context 亦有 byte drift，且不同 response
arm 的後續 request sequence 不保證相同。不要放寬 hash、猜 hash、手修 loop heads、改 production
prompt、換題，或作 population/timing claim。

Canonical full handoff:
`<remote-home>/cpachecker-experiments/runs/issue166_semantic_precision_census_20260901/NEXT_AGENT_HANDOFF.md`。
