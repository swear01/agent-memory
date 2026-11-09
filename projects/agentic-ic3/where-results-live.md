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
  - 同一個 DeepSeek Flash，kernel 改良把 31 題從 4 題解出提高到 17 題；關鍵是 UNSAT 回覆整組檢查（Houdini，issue 008）和 first-cycle guard。逐條檢查會擋掉要一起才 inductive 的 lemma（lrg pairwise、fifo per-slot）。
  - lemma 語言沒有量詞／index 變數；issue 001 加了 `X[e]`，issue 010 family expansion（模板＋實例化＋Houdini）在 p27a benches 從 6/16 提高到 12/16。
  - 離線分析：13 個未解 run 中有 6 個的候選池其實已含完整 invariant（kernel 損失，不是模型失敗）；wrap 的位寬算術 lemma 在低 frame 會被默默接受（issue 016 width lint）。
  - 2026-11-09 Opus 5.5 medium pilot：single_adder_ws_miter、nerv-pc-bwd 首次 SAFE（rIC3 3600 s TO），但比較對象是舊 kernel 的 DeepSeek run，還不能歸功於模型。
- NeuroAbs `develop`（到 PR #33）另有 adaptive split flow：LLM 選 index 拆分、timeout 時看 rIC3 lemma 摘要決定 resplit/abstract；結論是 split 有效、abstraction 至今沒有可測收益。
