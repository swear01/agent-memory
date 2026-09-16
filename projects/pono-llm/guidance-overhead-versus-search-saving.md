---
title: "LLM 引導效益須計入請求開銷"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: c4ffe6463f2da8563ce4836d4bf0f244766101558bc4ba359e16ffede28d6201
---

# LLM 引導效益須計入請求開銷

歷史助手回報某個 fib 案例 baseline 約百秒、guided 逾時，並指出 Stage 2 的多次模型請求可能抵消減少 CEGAR rounds 的收益。

同時對照 solver 工作量、模型等待與總 wall time。候選更多或 refinement 更少，不足以證明端到端加速；先量測再改觸發策略。

此片段未提供分段時間原始量測，不能確定所有時間差都來自 Stage 2；也不能據單一 bit-level 案例宣稱模型永遠做不到。跳過 Stage 2 仍只是當時的提案。
