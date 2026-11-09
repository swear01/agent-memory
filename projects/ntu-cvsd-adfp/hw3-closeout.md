---
title: CVSD HW3 繳交與 DC 面積最佳化收尾
scope: projects/ntu-cvsd-adfp
status: v1 submitted-and-verified; 11-09 exploration ongoing
updated: 2026-11-09
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

## 11/09 架構探索（目標 A×T 再降 10%，截止 11/10 13:59）

- 目前最佳 k4r2e：551,679 @ 2.29 ns，A×T 1263.35（比 v1 好 1.2%），gate sim 已過。尚未進 repo，進 repo 需使用者同意。
  共用資料夾在課程 repo 的 `hw3_explore/`（README、`results/SUMMARY.txt`、`results/rootcause_20h20.txt`）。
- **multicycle 放寬探針只能指出壓力在哪，省下的面積拿不回來。** 只放寬進 uh/ul 的 f_u 級可省 38.5k，
  但所有真的在 RTL 切開 f_u 的做法都變差：加一級複製暫存器 +20k；兩個部分和 +20k；carry-save 單元暫存器 600–640k（+50–90k）。
- **找面積變大原因的方法：** 在 run 目錄用 `read_ddc` 輸出每顆 cell 的名稱、ref、面積，再依名稱群組與 ref family 對兩個 run 做差。
  結論：DC 面積主要取決於兩件事。一是算術是否留在 DW `+` 區塊內（carry-save 會把它打散成 U*/ACHCON 散閘並讓暫存器變寬）。
  二是時序逼出多少高驅動 cell：把邊界邏輯併進同一 cycle 的 cycle cut 只省 0.3% cycle，卻在 window/esel/f_u 附近多出約 24k 的 DFFHQ/NAND2。
- **SRAM Q→raw_r 失敗的根因：** DC 對 raw_r 做相位反轉（變成 DFFSX4/DFFSRX4 再加反相器），或選用 D 端負載大的 DFFRHQX8。
  compile 之後才 size_cell 救不回來。在第一次 compile 之前加
  `set_dont_retime [get_cells -hier raw_r_reg*] true; set_register_type -exact -flip_flop DFFRX4 [get_cells -hier raw_r_reg*]`，
  compile 後再加 dont_touch，邊界就會 MET，面積只多 ~3k（屬於 DC 每次跑的雜訊）。
- 同一設計在 DC 上 run-to-run 差 ±3k。比較候選時要在相同 period 下比，差異小於 ~5k 不算訊號。
