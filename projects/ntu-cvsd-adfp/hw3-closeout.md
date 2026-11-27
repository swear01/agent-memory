---
title: CVSD HW3 繳交與 DC 面積最佳化收尾
scope: projects/ntu-cvsd-adfp
status: v1 submitted-and-verified; 11-09 exploration ongoing
updated: 2026-11-09
---

- **同一份 syn.tcl 重跑面積不同。** 先讀 `syn.log` 確認 `dc_shell` 或 `dcnxt_shell`。vACDEU40 的交件結果是 `dc_shell` 約 561,030 area、2.28 ns；dcnxt_shell 同樣 MET 但約 566,614。兩臂用同 period、相同工具與 flow；當時重跑變異約 ±3k，小於 5k 的差異不先當收益。於目標機器重跑 DC 和全部 gate benches，核面積與 log。
- **multicycle 放寬探針顯示很多面積，改 RTL 卻變大。** 探針用來找壓力位置，不能預估真實可回收面積。用 `read_ddc` 匯出 cell 名稱、ref、area，依名稱群組及 ref family 做差；看拆掉 DW 算術後的散閘、額外 registers，及時序推高驅動 cell 的影響。探索中的 k4r2e gate sim 通過，沒有取代原 v1 繳交。
- **SRAM Q→raw_r slack 差，compile 後 size_cell 救不回。** 查 DC 是否選了反相儲存元件加反相器或大 D 端電容。原驗證把 `raw_r_reg*` 在首次 compile 前標為 `set_dont_retime`，`set_register_type -exact -flip_flop DFFRX4`，compile 後再保護該邊界，該 run 恢復 MET。用 timing path 與 gate sim 檢查，變更不自動套用到別的 library 或架構。
- **ADFP home 和 COOL 包大小不同。** ADFP 含 netlist、SDF、ddc、area／timing reports，COOL 依當時 TA 回覆可省相應檔。用各自官方規格、解包雜湊與 COOL receipt 核對。PR merge 與後續探索都不代表送出了新版本。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/ntu-cvsd-adfp/hw3-closeout.md)
