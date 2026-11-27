---
title: CPAchecker #170 CFA-native precision compilation
scope: project
project: cpachecker
tags: [vguide, cegar, predicate-abstraction, precision-compilation, cfa, issue170]
status: deferred
created: 2026-09-03
updated: 2026-09-21
---

- **看到大型 precision compiler、IR 或多 backend 計畫。** 使用者已取消把這條擴建當研究主線；保留既有 C0–C2 實作與 C3b 證據，#173 utility 暫緩。若未來有明確隔離比較支持，再另立小範圍問題。現行研究入口見 [研究主線](vguide-predicate-research-roadmap.md) 與 #182。
- **從 observed path 的 guard transport 得到 predicate。** PR #171 的 frame transport 使用 authoritative `ARGPath.getFullPath()`、native taken `CAssumeEdge` formula、declaration-backed MemoryLocation 和 exact CFA-node placement。只有 `FV(g) ∩ MayDef(tau) = ∅` 可產生 path-preserved `PRECISION_ONLY` 維度，不能上升成全路徑不變量或 `ENTAILED`。跨函式、未知呼叫或寫入、partial definition、無法對齊的宣告／path hole 都應有明確拒絕原因。合併 reached precisions 時先收 distinct precisions 再 union。
- **Compiler dump 有 candidate，actual precision 卻沒增加。** #181 的 `nested3-1` 有 84 筆 TRUE no-op，consumer 明確拒收 TRUE。分開計候選、真正新 precision 和 exact-head `precision_local_after` 讀回；注入前先記錄 stock native delta。C3b 在凍結五題恢復 5/5，零模型呼叫；partial C4 僅 `hh2012-ex1b` 有 consumer-positive 證據。不能從 replay 推整體 usefulness、timing 或泛化。
- **把 round cap 當 HTTP attempts 硬上限，或把 first round 當 residual learning。** `LlmCallScheduler.recordCallCompleted()` 舊邊界只計成功回合；失敗 request 另核帳。在作出硬上限承諾前驗 attempt cap，或用證明有界的單一路徑。當輪 compiler candidate 若在 call 後才注入，必須檢查下一份 prompt 才能說有 context exposure。
- **#174 解了就重開 #173。** MathSAT fix 已合併 #176，固定舊 #173 STOP packet 仍保留；要補 formal matrix 或前進 C5，另行 preregister 並預建驗證當前 runtime。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/cpachecker/precision-compilation-issue170.md)
