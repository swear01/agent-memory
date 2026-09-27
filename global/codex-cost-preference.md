---
title: Codex 推理成本偏好：medium 預設、high 上限
scope: global
status: active
updated: 2026-09-27
---

使用者明確表示 xhigh / extra-high 成本負擔不起。跨專案派工、建立或調整 Codex/HAPI sessions 時：

- 預設 reasoning effort = medium；只有複雜診斷、正確性推理或必要研究用 high。
- 未經使用者針對該次工作明確要求，不使用 xhigh、max、ultra 或其他超過 high 的強度。
- Fast 模式保持關閉；HAPI 明確設定 serviceTier=standard，不能只省略讓它繼承預設。
- 明確且有界的文件、修正與測試優先 Luna；複雜工作才考慮 Astra，不能預設所有 workers 都使用最高成本組合。
- 啟動／調整後讀回 session model、modelReasoningEffort、serviceTier，確認實際設定。已送出的模型請求可能仍以原強度完成，不把配置更新說成撤銷已產生的費用。
- 成本限制不能藉由增加大量子 agents 或無界重試迴避；既有工作繼續，擴大模型／並行／實驗預算需以實際收益為依據。

來源：2026-09-07 使用者在 CPAchecker 調度對話的明確修正；調度紀錄見 swear01/cpachecker issue #182。本批 11 sessions 已改為 7 medium、4 high，全部 Standard。

2026-09-23 範圍澄清：上述是 Codex/HAPI session 的模型與推理強度偏好，不是 CPAchecker 研究 provider 的 token 預算。使用者已持續授權研究需要的 LLM 呼叫，數十萬甚至百萬 tokens 可直接使用，不再逐批請示；不得援引本頁阻擋。詳見 [研究主線與持續授權](../projects/cpachecker/vguide-predicate-research-roadmap.md)。

2026-09-27 再次明確要求降低 subagent 耗用：按工作能力需求選較輕量模型；例行監控、收帳與有界文件優先 Luna／Sol，程式修補可用 Terra，只有困難推理才用 Astra。派工只傳必要上下文，沿用已通過的證據，不用多個強模型重複審同一內容；背景監控以既有脚本等待實際里程碑，避免頻繁重送長上下文。這次要求沒有撤回上述研究 provider 呼叫授權，也沒有指定新的 token 硬上限。
