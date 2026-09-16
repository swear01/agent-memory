---
title: "研究 baseline 與競賽 witness 合規要分開判定"
scope: projects/cpachecker
status: active
updated: 2026-09-16
evidence_digest: 8b545e69bc97d6c64dc2f4cb609f362d48782c051d1c3f74bafde8545853604b
---

# 研究 baseline 與競賽 witness 合規要分開判定

歷史助手承認把較緊的競賽 witness validator gate 當研究 baseline acceptance 過度排除資料，將大量 timeout 改列 validation inconclusive，同時保留單一錯誤結果隔離。

按研究目的與資源記 measurement、correctness 和競賽合規，不把驗證 timeout 當 wrong，也不反過來當 correct。來源對 baseline 可用性的結論及後續 dataset 分層是回報／計畫；更寬 budget、重跑、文件更新與所有案例正確性尚未由此證明。
