---
title: 邏輯合成一律用實驗室商用工具（Synopsys DC／Cadence Genus），不要用 Yosys
scope: global/preferences
status: active
created: 2026-11-03
updated: 2026-11-03
tags: [synthesis, synopsys, design-compiler, cadence, genus, yosys, eda, preference]
---

# 合成工具偏好

使用者明確且多次要求：需要把 RTL 合成成網表（包含給 CULS 或 ABC 做 AIG benchmark）時，**一律用實驗室伺服器上的商用工具**，不要用 Yosys。

- Synopsys Design Compiler：在新的 bash login shell 執行 `source /apps/eda/synopsys/synthesis.sh 2022.12-sp6`，然後用 `dc_shell -f script.tcl`。
- Cadence Genus：`source /apps/eda/cadence/genus.sh @ver` 列出版本，再 source 指定版本。要的是 generic gate 時，用 `syn_generic` 和 `write_hdl -generic` 最快，可設 `max_cpus_per_server` 開多執行緒。
- 五台 Ubuntu 主機（mazu、cthulhu、athena、valkyrie、zeus）都掛 `/apps`。不要印出或 cat license 檔，也不要動 `/apps`。
- 網表轉 AIG 只能用非合成類的轉換：GTECH 或 generic 元件改寫成簡單閘，再用 ABC `strash; comb; write_aiger`。
- 交辦子 session 時，要把這條規則寫進 brief。2026-11-03 有子 session 又改用 Yosys，被使用者糾正。
- 舊紀錄 [[tools/yosys/large-rtl-to-aig]] 是 2026-10-30 用 Yosys 做的，只能當歷史參考，不要再照那個流程做新的合成。
