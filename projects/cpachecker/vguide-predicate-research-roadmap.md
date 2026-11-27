---
title: CPAchecker VGuide predicate 研究主線
project: cpachecker
scope: project
tags: [vguide, predicates, cegar, nested-loops, research]
status: active
created: 2026-08-26
updated: 2026-10-21
---

## 研究授權與驗收

- **被舊 packet 的模型用量上限卡住。** 使用者已授權 CPAchecker 研究按需主動使用 LLM，數十萬乃至百萬 tokens 可接受。舊 #269／#278 的限制專屬該包，既有 STOP slots、#197/#215 correctness holds、Reserved44 與 frozen 證據仍有效。新研究先寫假說、比較、分母和停止條件，再直接執行。正式模型呼叫走 [既有 production transport](vguide-experiment-transport.md)，保留完整原問答、失敗、usage 與實際 consumer 證據。
- **單題有效或替代集合進步就要收尾。** 三條路線分別追蹤，若有複現改善就擴至各自完整 set；無改善先定位實作、表示、consumer、生成、證明或資源限制。共同分母 hard218＝224 固定父集扣原 6 個 infrastructure censor；另有完整 array-cav19 13 題、sorting 指定 6 題與完整命名盤點、reducercommutativity 50 YAML（28 unreach-call，22 其他 property）。保留 new/lost/wrong/UNKNOWN、coverage、整體 CPU/wall、模型成本；未套用不是失敗或成功。預先鎖模型與 fallback，事後最佳 union 不是等成本 portfolio。正式比較以整服務 600 CPU／900 wall 判，包含 gate、生成策略與必需的精確 validation，未知用量另列。
- **合併後已有 native C 格式，卻仍無候選入場。** BMC fixed-bound 不觸發 Predicate CEGAR spurious hook；`predicateAnalysis-linear.properties` 曾證明 INTEGER 可以接 production native consumer（10 候選／12 exact placements），兩臂仍 UNKNOWN。PR #293 保留 SMT trace heads，native 另分 eligible heads 與 machine model；勿把 native 集合用於 SMT。區分 source validity、精度位置、實際注入與結果，分開驗 native output 與 adapter predmap。

## 三條路線的可用證據

- **#281 轉換後 Stock 已可解，別歸因模型。** pair-symmetr2 的原 global bound 讓舊原題 eligibility 為零；保守辨識不變 global 後同 N100000/ILP32 原題兩次 TRUE，paired Stock 兩次耗盡 CPU，轉完沒有合法 loop head，0 模型呼叫。strict endpoint 的 `N-1` 曾在 INT_MIN wrap 造成錯 TRUE；以 BigInteger 計算並確認 index、RHS、endpoint 都可表示，保留兩端反例控制。array-cav19 原樣回答的完整 13 題能力評估 5 TRUE（3 題轉換直解、2 題實際用 LLM）；生成成本另列，非等成本 portfolio。若 plain C bytes 不變而 replay miss，檢查 `source.file` 路徑及完整 Java body。
- **#282 Frama-C/WP 所有通過，但沒有初始化義務。** Frama-C31 要顯式 `-rte-initialized`，核對原 C、所有函式及 full obligation set。舊 F1 bubble/selection 省略此條件，附上後 raw plan 有未證 init，不能據舊 TRUE 推原 C。後續完整 auxiliary 23 題 F0 全 UNKNOWN，E3 5 TRUE；原答零新 call 固定確認包含 init，仍非 CPAchecker 增益。VLA alloc 的 freshness、bounds、nonalias 與 lifetime 不能借 scalar nondet 初始化代替；未證義務保持未證。
- **#287 INTEGER 得到 TRUE 但還未接回原題 C。** 需要同題 exact overflow/cast／signedness 等前置義務及原 property 接合。cast UF 和 signed overflow gate 不涵蓋所有 mixed／unsigned 或 pointer，已有反例。25 適用題同策略 F0／v5 原答／v6 修補為 7／12／15 TRUE、各 5 FALSE；8 個新增／退步確認題讀回三臂，v6 八個新解可重複，部分修補有人工概念提示，確認並非 fresh generation。hard218 中 7 題適用，F0 解 2，FIRST1／EVERY1_HISTORY 各 5，同增 `avg10-2`、`sum10-1`、`sum20-2`；211 未套用。歷史無 LLM v1 也曾解此三題但配置／預算不同；新研究需加同流程 F0 及範圍分析或模板控制。先跑 INTEGER 的未接合 TRUE 只能當排序訊號，連未證與不合格題都留在分母。
- **#296 Step Context 數字不能拿 #287 補。** 195 題三臂 F0／Existing／Step＝16／14／15；Step 對同批 F0 新增 0，對 Existing 單次多解 `iprotocol.1` 未於兩次 fresh 確認重現。frozen replay 限制見 [provenance](vguided-experiment-provenance.md)。
- **關係在 trace 中有效，不等於模型穩定選對。** `hh2012-ex1b` delayed oracle matched replay 從 UNKNOWN 到 TRUE；#149 Muse answer-free nested-loop cue 的 blind A/B 為 final hit 0/3→2/3，最低編號完整 treatment replay 兩次 TRUE。這支持該例 mechanism，不能外推命中率。需要因果歸因時，用完整原始 response 對 suppressed injection，再按獨立證據移除群組。
- **關鍵路徑最終號稱成功但中途結束不完整。** 原 C proof 採納需驗正常退出、完整 request/response/plan receipt 及 verified obligations；timeout 中已印的 TRUE 保留 raw 但不採納。cgroup OOM 有可信 supervisor/identity 對應才分類資源 UNKNOWN，terminal 缺失仍獨立記。ArrayAbstractionAlgorithm 的 delegate `AlgorithmStatus` 要合取且尊重 `stopAfterError`；imprecise CFA 回原 C 重驗的 status replacement 不同。PR295 對 075 同 compiler 恢復正確 FALSE，屬 wrapper 修正。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/cpachecker/vguide-predicate-research-roadmap.md)
