---
title: NeuroAbs keep-or-X decision 實驗（PicoRV32、rIC3）的已驗證結果與卡點
scope: project
project: rtl-relate
status: active
created: 2026-10-22
updated: 2026-10-25
tags: [neuroabs, picorv32, ric3, cegar, decision-model, gpt-6-luna, deepseek]
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
- 2026-10-23 診斷：TIMEOUT 的原因是 rIC3 版本，不是模型。論文用的是 HWMCC'24 版 rIC3，跑同一個 B0 BTOR2，中位數 26.3 s（kind 在 depth 21 完成證明）。1.5.2 吃 BTOR2 時，portfolio 沒有 bit-level kind（詳見 [rIC3 portfolio 組態](../../tools/ric3/portfolio-config-by-version.md)）；加上 `-e kind --step 1` 只要 8 s，master `8c6df1b` 的 portfolio 也要 70 s。作者的 `problem.aig` 帶 `-formal`，多 23 個 PicoRV32 內部 assert 和 2 個 `restrict`（56 bad）；`problem_new.aig` 不帶 `-formal`（33 bad），與 B0 對應。兩者都用 `-zinit`，沒有遺失 init。`design.aig` 是空檔。HWMCC24 版只檢查 bad[0]，所以作者的 AIGER 在該版 0.2 s 即完成，這不是論文的時間。完整紀錄在 `<remote-home>/research-archives/ric3-repro-20261022/`。
- 2026-10-24 B0／B1 計時（master `8c6df1b`、`taskset -c 16-31`、一次一個、各 3 次，54 次全部 UNSAT；PR commit `4aa4ad9` 的 README 有完整表格）。B1/B0 中位數比：SLTI 0.64、AND 0.82、BGE 0.93、SH 1.01（B1 沒有剩下 X，可當雜訊對照）、ORI 1.02、ADD 1.04、SUB 1.06、XOR 1.06、ADDI 1.07。B0 中位數在 103–202 s 之間。全 X 並沒有讓 rIC3 普遍變快。不綁核的那一輪雜訊太大（B1/B0 0.55–1.38），只留作對照，見 [[machines/mazu/cpu-timing-ecore-pinning]]。
- 2026-10-25 DeepSeek V4 Flash（gateway、effort low）D1 生成／D2 keep-or-X／D3 keep-or-X＋COI 九題全部建好並計時（81 次 UNSAT；PR commit `436ed20` 有完整表格）。相對 B0 的幾何平均：B1 0.95、D1 0.98、D2 1.05、D3 0.97；每個 decision arm 都是 4 題贏 B1、5 題輸，沒有比全 X 好。只有 D2 全部由 1017 回答，D2 最慢（SH 是 B1 的 1.44 倍）。D1 每題有 7–20 條 unsound（被 checker 擋掉），keep-or-X 不會有。CEGAR 後只剩 12–15（D1/D2）、2–10（D3）個位置。List price 約 $28（D2 約 $0.45／題，D1、D3 約 $1.3／題）。D1、D3 有部分答案被 gateway fallback 到 0731（見 [[projects/deepseek-latch-gateway/fleet-routing-and-failback]]）。 使用者 2026-10-25 決定不重跑 D1、D3，接受目前的模型混合。

## 卡點與待決

- Luna 第一個正式請求就收到 opencode Go 週額度 429，`Retry-After` 到 2026-10-26 08:00 台北時間；A1–A3 未跑，沒有改用其他模型。見 [opencode Go 直連](../../tools/opencode/go-responses-api-direct.md)。
- ~~在 rIC3 1.5.2 下 PicoRV32 B0 不收斂~~：已查明是 1.5.2 BTOR2 portfolio 的組態問題（見上）。使用者 2026-10-24 決定：用最新的 master `8c6df1b` 和預設 `bl_default`，不為單一題改用 `bl_hwmcc`，所有數據自己重測。
