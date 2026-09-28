---
title: "多路研究的 comparator 與確認範圍必須分層"
scope: "projects/cpachecker"
status: active
created: 2026-09-28
updated: 2026-09-28
---

# 多路研究的 comparator 與確認範圍必須分層

CPAchecker 多路實驗若同時有 Stock、零模型控制（F0）、模型策略與不同 verifier backend，`NEW`／`LOST` 必須連同 comparator、母體及 backend 一起記錄。相對 Stock 的 F0 gain 是 configuration gain；模型策略相對 F0 的 gain 才能支持模型增量。輔助 WP backend 的證明不能改稱 native CPA predicate-injection gain；跨 backend 去重後的 task 數只能稱研究覆蓋。

確認集合也要依每個實際比較建立，不能只確認模型策略相對 F0 的差異，就假定 F0 相對 Stock 的差異已確認。每個 observed NEW／LOST 應保留原結果，再以獨立執行確認；固定 response replay只證明 response-to-outcome 可重現，不證明新的 provider draw 會產生相同 response。非重現結果仍留在原始觀察，最終結論要分開列 observed 與 confirmed。

資源終局若是 clean `UNKNOWN`，應保留具體原因。例如 verifier interpolation failure、記憶體上限或 CPU 上限不能統稱 infrastructure failure，也不能因為不是 wrong verdict 就算成功。凍結摘要若有 metadata 誤植，但 registry、官方 YAML、argv 與實際 `UsedConfiguration.properties` 一致，保留原 frozen hash，另加 hash-pinned correction sidecar；不要靜默改寫或重跑正確的 scientific execution。

完整 service 成本需檢查 `budget_compliant`，不能只讀 verifier UNKNOWN 或 `timing_invalid=false`。#296 的一筆 fresh confirmation 在 watchdog checkpoint 599.543116 CPU 秒後，因收尾到達 final 600.095 秒；600秒契約下必須保留 INVALID。固定0.5秒 reserve 不是硬上界；prospective headroom 修正需用真正 worker（包含 post-run provenance hashing 與 receipt）測 final service totals，單獨測 watcher／kill 不涵蓋該成本。修正後的獨立對照須保留原 invalid 與整個受影響配對，明列 replacement 原因，不能覆寫原始結果。證據：issue #304；`<experiments-root>/reports/issue296-confirmation-20260928/closure-fresh/REPORT.md`。

多階段 native control 的 HTTP transport 紀錄可能在 `overflow/console.log`、`integer/console.log`，不只 `cpa.log`／`service.log`。以同一 execution directory 的事件 multiplicity 做去重後，#296 pilot-D 的 HTTP／payload／round 可核對138個完成回答；原先只讀前兩種 log 會漏掉30個控制組呼叫。固定回答 replay 的 usage 是原 provider 回答攜帶的歷史成本，不能再算成新 API tokens；replay CPU/wall 與原 live generation 成本分列。
