---
title: PicoRV32 多數 assertion 行為候選與無 Formal 階段
scope: project
project: rtl-relate
status: active
updated: 2026-10-01
---

- **面積或多數 bounded PASS 被當原版行為涵蓋率。** 固定 PicoRV32/riscv-formal、wrapper/defines 和 86 個 assertions，cover、手寫 sby、相關 tasks 分開。ALTOPS 簡化 M 算術；generic cells、真實算術 mapped area、RVFI/ALTOPS mapped area 和 solver 時間不可互換。`RISCV_FORMAL_BLACKBOX_REGS` 是 upstream 機制，欠跨指令 register RAW；DC 不能綜合 `$anyseq`。82/86 bounded PASS 中仍有已刪 minstret 的案例，需核非 trap 和 assertion precondition covers。
- **刪 M/PCPI 以後散出 CSR trap，先怪單埠 register file。** 以只刪 M 和只換單埠兩個控制定位。固定 `WITH_PCPI=0` 會改未知指令的 wait/timeout，使部分 trap 提早進 RVFI。`csrw_mcycle_ch0` 有第 15 拍反例；另一條 CSR 同拍仍可達。另有模型實作把 trap 的 rs1 address/value 清零，使未對齊 read/write 的 checker 重算錯；原樣模型、格式恢復及人工修理分開記。
- **Jasper / SBY 原版也開始報假反例。** `expect pass,fail` 可使 SBY FAIL exit 0，讀 task status。Jasper 匯入同 checker/wrapper/defines，用 `assume -constant checker_inst.<signal>` 保留 `rvformal_rand_const_reg` 的任意但全 trace 固定，涵蓋 `insn_order`／`register_index`。漏掉時六個 register/PC/order 原版題假 FAIL。固定條件的原版 86/86 bounded、候選 05 72/86；14 FAIL 包含 M 和 CSR，不是等價證明。重跑入口為 `experiments/picorv32_majority/`，不要把一般 CI 說成候選正式驗證。
- **候選 FAIL 得較快，或並行從 4 改 6 就報 solver 加速。** 核同題同結果、固定並行，相同定義的 solver 與 session time，另外報失敗／timeout／未落盤。現有單次類別比較與少量配對不足以宣稱穩定整體加速。使用者已暫停新呼叫和改版；原 raw logs 非全部在 Git。先讀 repo 及原收尾，勿依歷史下一步自行再跑。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/rtl-relate/picorv32-majority-model-2026-09-30.md)
