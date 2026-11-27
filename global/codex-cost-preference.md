---
title: Codex 推理成本偏好：medium 預設、high 上限
scope: global
status: active
updated: 2026-11-28
---

- 派工或新建 Codex／HAPI session 時預設 `modelReasoningEffort=medium`。複雜診斷、正確性推理或必要研究用 `high`。高於 `high` 的強度須由使用者針對當次要求。Fast 保持關閉，HAPI 明確送出 `serviceTier=standard`，避免省略後繼承上次選擇。
- 卡住先用現有證據和有界重試。例行監控、收帳與有界文件優先 Luna／Sol，程式修補可用 Terra，困難推理才考慮 Astra。派工只傳必要上下文，背景監控用現有腳本等里程碑，省去重送整段上下文。
- 啟動或調整後讀回 `model`、`modelReasoningEffort` 與 `serviceTier`。設定生效後，已送出的請求仍可能按原強度完成。
- CPAchecker 研究 provider 的 token 預算有[獨立的持續授權](../projects/cpachecker/vguide-predicate-research-roadmap.md)，可依研究需求呼叫，無須逐批重問。

依使用者 2026-09-07、09-23、09-27 的成本與研究範圍修正整理。
