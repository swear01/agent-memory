---
title: CVSD HW3 繳交與 DC 面積最佳化收尾
scope: projects/ntu-cvsd-adfp
status: v1 submitted-and-verified; grading pending
updated: 2026-11-05
---

# HW3 完成狀態（2026-11-05 v1）

- 繳交 s31c RTL，2.28 ns，flow vACDEU40；Total cell area（含 SRAM）561030.071656，slack 0.00，A×T 1279.15。Gate sim tb0–6 於 2.28 ns 全過（cycles 1035/1035/1036/15368/1039/15371）。
- ADFP home：`<remote-home>/cvsd_hw3_v1.tar`（`r14k41044_hw3/` 12 檔，含 netlist/SDF/ddc/area/timing report）。解開後 md5 與 repo 一致。
- COOL：同名 tar 只放本機 7 檔（TA 回覆 netlist/report 可省）。經已登入瀏覽器的 Canvas API 上傳，下載回來 SHA-256 相同，狀態 submitted、未遲交。
- Course repo PR #20 已 merge；`02_SYN/02_run.dc` 還原成 starter 的 `dc_shell-t`。

## 可重用經驗

- **dc_shell 與 dcnxt_shell 對同一份 syn.tcl 給不同結果。** vACDEU40 在 dc_shell 得 561030，dcnxt_shell 得 566614（兩者皆 slack 0、gate sim 過）。重現既有結果前，先看該 run 的 syn.log 標頭（"Design Compiler Graphical" = dc_shell）確認工具，再用同一個 binary 跑。TA 評分用 starter 的 `dc_shell-t`，所以最終 flow 應以 dc_shell 驗證。
- 對面積有效的 flow：首輪 compile_ultra -retime 時 set_dont_use DFFHQ*/ADDFH*/CMPR*X4，之後解除 dont_use，再做大量 `compile_ultra -incremental -retime` + `optimize_netlist -area` 交替，最後用 `compile -incremental_mapping -map_effort high` 收 timing。
- 打包前在目標機器上重跑 DC＋全部 gate tb，並比對面積完全一致，才算可交。
