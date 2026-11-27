---
title: "LLM predicate 的回饋缺口與不變量學習邊界"
scope: "projects/cpachecker"
status: active
updated: 2026-09-28
---

- 提議新回饋前先讀 production prompt。Native75 已有多輪排程、最近 4 輪 CE/outcome、precision 累積和 source slicing。曾看到 `CeSummaryBuilder` 每條 relation 截 280 字元、structured CE 缺 branch_conditions／ssa_values／assignments、infeasible pivot／ARG prune unavailable；source 仍可含分支和賦值，`G ! overflow` 亦有提供。模型增量與注入前 `native_delta` 分開。當次 usefulness gate disabled，raw whitespace 去重不等於 solver 等價。
- PRECISION_ONLY 不必是 invariant。ICE 正反／implication labels 需真 teacher；整條假反例 UNSAT，局部 SAT prefix/transition 未必可達。用實際 ARGPath 核完整 CFA edges 到達 prompt，另驗 consumer 和解題效果。#152 RETIRED，不重跑停止格；新研究凍結母體、answer-free 與等長 irrelevant controls，context／scheduler 分開消融。
- 拆 reducer 做 direct baseline 時保留 corpus 共同 C 語意如 `cpa.predicate.memoryAllocationsAlwaysSucceed=true`，各題依 YAML 保持 ILP32/LP64。漏帶此設定曾把 malloc 允許返回零的反例記為 WRONG，修復後仍可因插值得到 UNKNOWN。
- `parsePureExpression` 未宣告 wrapper 遇 `~` 可殘留 `CProblemType`。ASTConverter 僅對 `op_tilde` 從有效整數 operand 套 integer promotion、保留 unary minus 行為，encoder 另遞迴拒未解型別。未處理的 exit 1 為 INVALID。
- PredicateCPARefiner 先 `checkCounterexample` 再進 VGuide；`fallbackWithoutInterpolation` 可能在 non-interpolating UNSAT 後仍重拋原插值錯誤。INFO 訊息不足以證 UNSAT。Source-only 無實際 block/SSA/pointer context 時以 `native_c_context_unavailable` 拒 `c:`。#302 的恢復入口驗真正 UNSAT 後只新增 PRECISION_ONLY，再用 `ARGReachedSet.removeSubtree` 重排；SAT 走真實反例處理。
- `recovery_precision_delta` 須扣 head 已有的 global/function/local，instance-only 不能視為整個 head 已有。用 CFA path hash、`firstGuardReason`、recheck 例外區分 `repeatedCounterexample` 與前置失敗。以 max rounds > 1 的固定重播排除 cap 混淆；該次重播仍 UNKNOWN。
- 診斷 jar 的 `config/specification/sv-comp-reachability.spc` 可能從 jar 路徑解譯，保留相對 `config`（可用 symlink），切 CWD 不夠。配置失敗維持 INVALID。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/predicate-generation-feedback-boundaries.md)
