---
title: Agentic-IC3 重現與改良的實驗位置與目前結論
scope: projects/agentic-ic3
status: active
updated: 2026-11-12
---

- **找研究進度只查 Pono 或 NeuroAbs。** Agentic-IC3 是「LLM 在 IC3 迴圈提 lemma」的重現與 kernel 主線。接手先讀 mazu `<remote-home>/agentic-ic3-repro/STATUS.md`、`results/gate-p*.md`、`results/analysis-*.md` 和 `issues/0xx-*.md`，查目前負責 session 再傳使用者決策；本機舊 HTML 不是原始 runs。
- **逐條 induction 都不過，就把問題歸因模型。** 同時檢驗互相支援的 lemma 集合；pool 已有完整 invariant 時另量 Houdini／kernel 損失。演算法允許固定寬度算術，要驗不會把 wrap 的候選錯當數學整數；量化 family 支援是不同表達能力。SAFE 由 pool 證出時要測去 IC3 的消融和等預算外部對照，才可歸因 IC3 kernel 或模型。
- **四題 gate 進步當 31 題全量。** 核 repository 當批 manifest、相同 cores／時間預算和完整公開 evidence；凍結 query/solver/source，分開 p15 全套與 p26 起短 gate。不同 kernel 的 Opus／DeepSeek 快照不適合作單變量比較；NeuroAbs 的 split 進度不可代替 Agentic-IC3 效果。受委任紅隊時依 [審查角色](red-team-review-role.md) 回報。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/agentic-ic3/where-results-live.md)
