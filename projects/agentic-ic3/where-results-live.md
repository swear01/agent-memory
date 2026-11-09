---
title: Agentic-IC3 重現與改良的實驗位置與目前結論
scope: projects/agentic-ic3
status: active
updated: 2026-11-09
---

# Agentic-IC3 實驗在哪裡

使用者的「LLM 在 IC3 迴圈裡提 lemma」主線是 Agentic-IC3（arXiv:2609.27162）的重現與 kernel 改良，不在 NeuroAbs 或 pono-llm。回答「跟我的研究有什麼關係」這類問題前，要先讀這裡；只讀 pono-llm／NeuroAbs 會低估進度。

- repo 與結果：mazu `<remote-home>/agentic-ic3-repro`（`results/gate-p*.md`、`results/analysis-*.md`、`issues/0xx-*.md`、`AGENTS.md`）；run 紀錄在 `/var/tmp/agentic-ic3-runtime/runs/`。本機 `<home>/Documents/agentic-ic3-repro/reports/` 只有 10/28、10/31 兩份 HTML 報告。
- 2026-11-09 時的重點（細節以 repo 為準）：
  - 同一個 DeepSeek Flash，31 題從 p3 的 4 題提高到 p15 的 17 題（每 run 30/62）；p15 的關鍵是 UNSAT 回覆整組檢查（unsat-set，c933e9a）和 effort medium。逐條檢查會擋掉要一起才 inductive 的 lemma（lrg pairwise、fifo per-slot）。issue 008（候選池 Houdini）在 p15 之後；p26 起的 gate 只跑 fifo_vis_48、lrg_48、fifo_ref_48、fifo_ref_64 四題，目前 main 沒有 31 題全集數字。
  - 對 rIC3（`results/ric3-baseline-2026-10-31.md`）：rIC3 1 h 解 10/31，但只給 1 E-core portfolio，很多是 600 s screen；p26 起每個 SAFE 都由 pool Houdini 證出，沒有「不用 IC3」的 baseline。2026-11-09 review（`results/review-clover-2026-11-09.md`）把這些缺口拆成 issues 017-024（全集重跑、預算對等 rIC3、IC3 貢獻消融、控制狀態列舉、提案逐條回饋、SAT reply 未檢查 lemma、gate 檢定力、opq 名稱依賴）。
  - lemma 語言沒有量詞／index 變數；issue 001 加了 `X[e]`，issue 010 family expansion（模板＋實例化＋Houdini）在 p27a benches 從 6/16 提高到 12/16。
  - 離線分析：13 個未解 run 中有 6 個的候選池其實已含完整 invariant（kernel 損失，不是模型失敗）；wrap 的位寬算術 lemma 在低 frame 會被默默接受（issue 016 width lint）。
  - 2026-11-09 Opus 5.5 medium pilot：single_adder_ws_miter、nerv-pc-bwd 首次 SAFE（rIC3 3600 s TO），但比較對象是舊 kernel 的 DeepSeek run，還不能歸功於模型。
- 對照文獻 IC3Syn（arXiv:2605.24619，TLA+，2026-11-09 讀原文）：admission 逐條（`Init ⇒ c`、`F_{k-1} ∧ c ∧ Next ⇒ c'`），沒有整組 Houdini；可以逐條成立是因為 LLM 直接寫帶量詞的 TLA+ clause，一條 ∀ 就涵蓋整個 index family。TLC 在有限實例上用全部可達狀態篩候選；clause memory 會重用；prompt 帶 known-true/known-false 清單與批次 bad states。評估：每題 600 s（兩題 2 h）、看不出有重複跑、沒報 token 成本、沒有污染對照（Endive 題目公開）。TLAPS 的無界證明主要由 LLM agent 寫，不是全自動。
- NeuroAbs `develop`（到 PR #33）另有 adaptive split flow：LLM 選 index 拆分、timeout 時看 rIC3 lemma 摘要決定 resplit/abstract；結論是 split 有效、abstraction 至今沒有可測收益。
