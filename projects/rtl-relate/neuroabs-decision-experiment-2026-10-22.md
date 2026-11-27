---
title: NeuroAbs keep-or-X decision 實驗（PicoRV32、rIC3）的已驗證結果與卡點
scope: project
project: rtl-relate
status: active
created: 2026-10-22
updated: 2026-10-26
tags: [neuroabs, picorv32, ric3, cegar, decision-model, gpt-6-luna, deepseek]
---

- Yosys 0.68 移除 `abc -fast`。公開 PicoRV32 template 先 `proc`、`delete t:$print`，輸出 AIGER 前 `setundef -anyseq`，`sim` 前 `chformal -lower`。作者 `add_input*` 將原始 'bx 變顯式輸入，該 setundef 契約下等價。
- NeuroAbs 原 SMT checker 缺 part-select／concat target、`|{...}`、`$signed` 及縮排 if；遇前端例外要先補表示、保留 rejected。CEGAR refine 的舊 port list 及重複文字可能再次覆蓋輸入，按原行和 word boundary 核對。
- rIC3 1.5.2 BTOR2 portfolio 缺 bit-level kind，PicoRV32 B0 需 depth 21。復現可用 `-e kind --step 1`，正式比較使用同 master、預設 `bl_default` 及相同綁核。HWMCC'24 版只看 bad[0]；比較 `-formal` 是否帶額外 assert/restrict、`-zinit` 和 bad 集合。rIC3 `--cex` 到限會在 lib.rs:147 panic，另記 timeout。見 [[tools/ric3/portfolio-config-by-version]]。
- 單一 branch keep-or-X 未減 COI state，九題 sample 的全 X 或 LLM arms 無普遍收益；模型 fallback 混合集已由使用者決定不重跑。PicoRV32 register file 的 `cpuregs_rs1/rs2` 可在 insn check 用 `$anyseq`、x0 仍 0，以 `check_implies` 門檻核對，此次 state 降且 rIC3 加速，保障條件依 property。
- Piccolo/Flute ILA 每個 GPR 在 refinement map assertion 中，不能全部 havoc。按鎖存 `__split_inst__` 拆 mem/xk.frame (rd≠k)/xk.rs1=r 或 xk.write，word 級 keep/word/havoc 及加法器 Ackermann UF `uf0`；rs1 split、UF、property split 缺一會失去該次 Piccolo ADDI 證明。先對照簡單結構規則，該次它勝過 LLM 與 auto CEGAR。Flute 的 `__IEND__` 可晚於文獻 BMC 深度，先驗 property 實際觸發，再解讀 bounded pass。
- Evidence 在 repo `results/decision_experiment/`、`results/fine_gpr_choices/` 及相關 archived runs。Luna A1–A3 當輪未執行，不能從 DeepSeek 結果代稱。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/rtl-relate/neuroabs-decision-experiment-2026-10-22.md)
