---
title: "LLM literal 排序仍須 solver 驗證及對照效益"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 213515e3f43e811a9befaae839ee058f4172b636d98cd2cc090c3b31165c2571
---

- **LLM 建議刪 literal 或更少 CEGAR rounds，被當成正確性／提速保證。** 將 LLM 順序當 heuristic，保留完整 IC3 frame／初態與 solver admission，少 literal 會擴大 cube 並加強否定 clause；reject 不能單獨解讀成可達。配對量 solver 工作、各階段請求等待及總 wall time，避免 Stage 2 多次呼叫吞掉收益。歷史 fib 單例 guided timeout 未有分段原始量測，不能歸因某階段；literal drop 順序本身也尚無 A/B 或實作證據。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/pono-llm/heuristic-order-not-performance-guarantee.md) · [請求與搜尋成本](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/pono-llm/guidance-overhead-versus-search-saving.md)
