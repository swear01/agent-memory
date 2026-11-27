---
title: AIsimpV 通用 decision model 調查與可替換決策點
scope: project
project: AIsimpV
status: active
updated: 2026-10-22
---

- **通用排行榜高分就拿來選 RTL abstraction。** JEVal 本次未涵蓋 code/engineering；各廠榜單混成本與延遲不可互換。使用者把決策延遲放寬到 2 秒、不跑本地，仍需就同一 RTL 訊號任務測簡單規則、同 backbone readout 和生成式推理。保留同 downstream solver/validator，驗保留率、實際 cost 和校準；未有可行 cut 的空間換選擇器也不會解題。
- **把整份 RTL 當選項特徵，或用模型信心當 soundness。** 先用 Yosys／COI 取 width、fanin/fanout、assertion 距離與 CEX 摘要，短選項拆成可觀察 decision。模型只選 keep/X、策略或 refinement 次序，正式 checker 檢查 overapproximation；不改 IC3 內部。AVR 已有自動抽象，需作 baseline；NeuroAbs 公開原始程式中 candidate/signal 來源含 Yosys constant propagation，不能把所有步驟都歸給 LLM。[NeuroAbs keep-or-X 研究](neuroabs-decision-experiment-2026-10-22.md)
- **Model list 有 Luna，但呼叫回 model_not_allowed 或 ModelProtocolUnsupported。** 當時 Go 直連 `/zen/go/v1/responses` 和 `x-opencode-session` 可跑 `reasoning.effort=low`；chat completions 不相容，`/v1/decisions` 是不同支援與端點。重新驗目前 config、endpoint、effective model/effort 和實際 accounting；free-tier CLI 會混入系統提示，不可當 controlled API。以同版 RTL、完整 output 或有效失敗、預先固定費用與效益讀回；截至本次資料，不能從定價或通用 benchmark 宣稱 RTL 加速。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/rtl-relate/decision-model-survey-2026-10-22.md)
