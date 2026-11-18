---
title: "Routine 推薦相關度下降的修法與研究主軸"
scope: projects/routine
status: active
updated: 2026-11-18
---

# Routine 推薦相關度下降的修法與研究主軸

- 使用者說「我的 routine」指 GitHub repo `swear01/routine`（每日 Discord 推三篇論文），不是 claude.ai 的 RemoteTrigger。
- 2026-11 相關度下降的原因：`MIN_PAPER_DATE` 不設上限，固定查詢一直挖舊論文；75 分門檻太寬；SAT/GPU 這類泛查詢帶進雜訊。使用者接受舊論文（backlog），不要加近期視窗。
- [PR #14](https://github.com/swear01/routine/pull/14)（`c7b08b9`）與 [PR #15](https://github.com/swear01/routine/pull/15)（`1cc59fb`）：門檻改為 `MIN_RELEVANCE_SCORE = 84`。研究主軸寫在 `RESEARCH_FOCUS`，選文 prompt 與查詢生成 prompt 共用。主軸是 LLM × formal methods 的兩個方向，soundness 都不依賴 LLM：A 是 LLM 加速驗證 engine（要對比非 LLM baseline），B 是形式工具檢查 LLM 產出。transition system／硬體 model checking 給 90+；其他 LLM × FM 給 84–89；Lean 數學定理證明和 autoformalization 最高 84 分。調整範圍時改 `RESEARCH_FOCUS`，不要只加關鍵詞。
- 加新方向時三處要一起改，否則論文到不了 LLM：
  - `is_relevant_candidate` 要求 `core` 與 `candidate_kind` 都命中，所以兩份清單都要加詞。避免 proof、specification、termination 這類泛詞。
  - 候選依 `POSITIVE_TERMS` 分數排序，只取前 30 篇送 LLM，新詞沒權重會被截掉。
  - arXiv 查詢裡沒加引號的詞，只會保留已知詞組，例如 `Dafny LLM` 會變成 `all:"llm"`。專有名詞要寫成 `'"Dafny" LLM'`。
- `workflow_dispatch` 的 `dry_run=true` 仍會上傳 `daily-report` artifact，但不發 Discord、不更新 Gist 已看清單，適合測試。`dry_run=false` 會發送並記為已看。
- 2026-11-18 在 `1cc59fb` 上驗證：[dry run 44317962414](https://github.com/swear01/routine/actions/runs/44317962414) 與 [正式 run 44319209110](https://github.com/swear01/routine/actions/runs/44319209110) 選出同三篇（LinguaArray、MASTA、FCP），使用者認可。同一篇兩次的分數差到 4 分（88→92），接近 84 分的論文進不進得了，可能每次不同。
