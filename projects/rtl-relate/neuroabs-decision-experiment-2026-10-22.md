---
title: NeuroAbs keep-or-X decision 實驗（PicoRV32、rIC3）的已驗證結果與卡點
scope: project
project: rtl-relate
status: active
created: 2026-10-22
updated: 2026-10-22
tags: [neuroabs, picorv32, ric3, cegar, decision-model, gpt-6-luna]
---

# NeuroAbs keep-or-X decision 實驗

承接 [decision model 調查](decision-model-survey-2026-10-22.md)。NeuroAbs fork PR #12（branch `feat/luna-decision-experiment`）加入 Luna 生成式 backend、keep/X decision backend（JSON 答案＋兩個機率，X 由程式改寫後仍走 Z3）、全 X baseline（B1）、`--candidate-scope coi`（A3）、transition-aware strict-OA checker 與實驗 runner。結果文件在 repo `results/decision_experiment/README.md`，run 目錄保存在 `<remote-home>/research-archives/neuroabs-decision-20261022/`。

## 已驗證

- 公開 PicoRV32 constant-prop 範本在 Yosys 0.68 跑不動：`abc -fast` 已移除（changelog 建議 `abc -g AND -fast` 改 `aigmap`）；`$display` 變 `$print` cell，`write_aiger` 拒收；殘留 X 也被拒。修法：`proc` 後 `delete t:$print`，AIGER 前 `setundef -anyseq`。
- `picorv32_input.v` 內 21 個 `add_input*` 是作者把上游 `'bx` 改成顯式輸入，在 `setundef -anyseq` 下等價，不是 LLM 抽象。
- NeuroAbs SMT checker 無法編碼 part-select／concat 目標、`|{...}`、`$signed`、縮排的 `if (...)`；ADD 全 X 的 232 次中 71 次因此失敗。原 `main.py` 遇到會直接拋例外中止，所以公開碼無法完整跑 PicoRV32。
- CEGAR `refine()` 在 PicoRV32 上會 assert 失敗：已還原的 input 仍在 port list，Pono dynamic COI 會再列出它；另有重複敘述文字造成的 input 被覆蓋（ADD 161 個 input 中 23 個）。已改成以 word boundary 檢查、跳過不在原行的 input。
- B1 九題全部建好：sound 100%，每題 2 個 no-op（上游 `count_cycle/count_instr <= 'bx`），CEGAR 1–5 次 refinement 後保留 0–17 個 X；SH 被全部還原。
- ADD pilot：B0 與 B1 在 rIC3 1.5.2 portfolio 上 69 分鐘都沒有 verdict（作者報 concrete 39.32 s，未重現）；kissat BMC 到 depth 25 兩者約 9 秒、無反例。riscv-formal 只在 cycle 20 檢查一次，bounded 探索無法區分 arm，難點在 unbounded proof。rIC3 時限問題見 [rIC3 portfolio 時限](../../tools/ric3/portfolio-time-limit-not-enforced.md)。

## 卡點與待決

- Luna 第一個正式請求就收到 opencode Go 週額度 429，`Retry-After` 到 2026-10-26 08:00 台北時間；A1–A3 未跑，沒有改用其他模型。見 [opencode Go 直連](../../tools/opencode/go-responses-api-direct.md)。
- 在 rIC3 1.5.2 下，PicoRV32 題的 B0 不收斂，以 rIC3 時間比較 arm 可能全是 TIMEOUT；需使用者決定是否改指標（如 proof 是否完成、或換有結論的題／checker 版本）。
