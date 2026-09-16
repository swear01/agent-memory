---
title: "LLM literal 排序仍須 solver 驗證及對照效益"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 213515e3f43e811a9befaae839ee058f4172b636d98cd2cc090c3b31165c2571
---

# LLM literal 排序仍須 solver 驗證及對照效益

使用者指出 LLM 尚無幫助；助手提出讓模型只提供 literal drop 順序、solver 逐步檢查，但又未經實驗承諾壞建議不比原本差。

將模型建議視為待測 heuristic，保留原 IC3 的 frame／初態等完整接受契約，不能從簡化偽碼宣稱 sound。較少 cube literals 會擴大 cube 並強化其否定 clause；拒絕也不單獨證明真實可達。來源僅設計，沒有實作或 A/B；solver checks 的成本與排序仍可能降低效能，不能保證有益。
