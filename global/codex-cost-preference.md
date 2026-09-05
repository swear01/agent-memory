---
title: Codex 推理成本偏好：medium 預設、high 上限
scope: global
status: active
updated: 2026-09-07
---

使用者明確表示 xhigh / extra-high 成本負擔不起。跨專案派工、建立或調整 Codex/HAPI sessions 時：

- 預設 reasoning effort = medium；只有複雜診斷、正確性推理或必要研究用 high。
- 未經使用者針對該次工作明確要求，不使用 xhigh、max、ultra 或其他超過 high 的強度。
- Fast 模式保持關閉；HAPI 明確設定 serviceTier=standard，不能只省略讓它繼承預設。
- 明確且有界的文件、修正與測試優先 Luna；複雜工作才考慮 Astra，不能預設所有 workers 都使用最高成本組合。
- 啟動／調整後讀回 session model、modelReasoningEffort、serviceTier，確認實際設定。已送出的模型請求可能仍以原強度完成，不把配置更新說成撤銷已產生的費用。
- 成本限制不能藉由增加大量子 agents 或無界重試迴避；既有工作繼續，擴大模型／並行／實驗預算需以實際收益為依據。

來源：2026-09-07 使用者在 CPAchecker 調度對話的明確修正；調度紀錄見 swear01/cpachecker issue #182。本批 11 sessions 已改為 7 medium、4 high，全部 Standard。
