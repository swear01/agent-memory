---
title: "多路研究的 comparator 與確認範圍必須分層"
scope: "projects/cpachecker"
status: active
created: 2026-09-28
updated: 2026-09-28
---

# 多路研究 comparator、配置與確認範圍

- 啟動前核對兩 arm 的官方 YAML、實際 argv 和 `UsedConfiguration.properties`，確認使用者指定的是單一 analysis（例如 `predicateAnalysis`）還是 portfolio。先查已完成對照。
- Stock、zero-model F0、模型與 backend 都要寫入每筆 NEW/LOST 的 comparator 和母體。F0 相對 Stock 是配置增量，模型相對 F0 才是模型增量。WP 輔助證明獨立列，跨 backend 去重數是研究覆蓋。Portfolio 要逐題核對 VGuide 的 predicate refinement hook 是否觸發、哪個 child 決定 verdict 及 stock 配對，configuration 賦值不足以歸因。
- 每個比較的 observed NEW/LOST 各自保留並獨立確認。固定 response replay 能確認 response-to-outcome，fresh provider draw 另測。clean UNKNOWN 保留 interpolation、memory、CPU 等原因。實際執行證據正確、只有 frozen 摘要誤植時，保留原 hash 並加 hash-pinned correction sidecar。
- 核 `budget_compliant` 以及整個 service 的 final CPU/wall。Watchdog 觸發後的 provenance hashing 與 receipt 仍耗預算，reserve 要用實際 worker 驗證。超限保持 INVALID；如做替代對照，同時保留原 paired 結果和原因。
- 多階段控制組還查 `overflow/console.log`、`integer/console.log`，以同 execution directory 的事件 multiplicity 去重，勿漏算 HTTP。Replay 使用量是歷史回答所載的 provider 成本，另計該次 CPU/wall。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/research-comparator-confirmation-scope.md)

合併自 [portfolio 歸因](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/portfolio-hook-attribution-boundary.md) 和 [單一 analysis／portfolio 核對](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/cpachecker/benchmark-analysis-config-identity.md)。
