---
title: Innovus Stylus physical checks per flow stage (check_drc after place needs -ignore_trial_route)
scope: domains/eda
tool: innovus
status: active
updated: 2026-10-21
---

# Innovus Stylus 各階段的實體檢查

Innovus 25.1 實測（U18 gcd，約 165 cells）：

- place 與 CTS 後直接跑 `check_drc` 會報上千個假違規（預設上限 1000），因為訊號線只是 trial route（Minimum Area、Out Of Die 等）。官方 `TCRcom/check_drc.html` 有 `-ignore_trial_route`，加上後兩階段都是 0。`-check_only cell` 仍會檢查 cell 對 trial route，還剩約 200 個假違規，不適用。
- 建議檢查點：floorplan 後 `check_floorplan`；powerplan 後 `check_drc -check_only special`（加 `check_power_vias`）；place／CTS 後 `check_place` 與 `check_drc -ignore_trial_route`；route 後 `check_drc`、`check_connectivity`、`check_antenna`，修正用 `route_eco -fix_drc`；filler 後 `check_filler` 再 `check_drc`；最後 `write_stream` 加 foundry deck（Calibre/Pegasus）sign-off。
- Stylus 文件在 `INNOVUS251/doc/UGcom`（User Guide）與 `TCRcom`（command reference）；`innovusUG`／`innovusTCR` 是 legacy 語法。Flowkit／Foundation Flow 文件沒有規定檢查點。
- Stylus 流程兩個常漏設定：沒放 IO pin 會讓 CTS 報 `IMPCCOPT-2215`（clk 圖不連通）且 `IMPOPT-665` 大量出現，`place_opt_design` 前加 `set_db place_global_place_io_pins true`；post-route hold 修正前要 `set_db timing_analysis_type ocv` 與 `timing_analysis_cppr both`，否則 `IMPOPT-7027` 擋下。
- `set_dont_use [get_lib_cells */PAT] true` 在 Genus 與 Innovus 都可用；`set_db ... .dont_use` 在 Innovus 會報 `IMPDBTCL-248`。

相關：[[eda-environment]]
