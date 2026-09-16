---
title: "候選 wrapper 層級須同步產生器、篩選器與去重器"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: a21af2d84194165d5a55d1e30f347e4a114ccf40987f7bd06fb7ce122706f59e
---

# 候選 wrapper 層級須同步產生器、篩選器與去重器

助手指出手造 eq 候選缺 predicate_ast wrapper，而合法 LLM 候選的 form 位於 wrapper 內；篩選器卻讀外層 form，連去重也讀錯層，回報零注入。

沿候選的產生、解析、篩選、去重與下游消費核對同一 schema。用實際候選結構驗證各段，不能把本地 wrapper 錯誤歸因模型不會產生候選。

來源只有定位與準備修正，沒有修後注入或 soundness 驗證；格式一致仍不代表候選不變式正確。
