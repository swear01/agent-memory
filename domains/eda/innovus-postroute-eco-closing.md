---
title: Closing the last few ps of post-route setup in Innovus — curated ecoChangeCell beats broad upsizing or repeated optDesign
scope: domains/eda
tool: innovus
status: active
updated: 2026-11-25
---

# Innovus post-route 收最後幾十 ps setup

CVSD 2026 期末 SCE（tsmc13，28 顆 SRAM macro，1.80 ns，margin-0 floorplan，密度約 91%）實測：

- **同一個 DB 反覆 `optDesign -postRoute` 會越跑越差**：從 −0.022 ns 的 DB 再跑，結果是 −0.05、−0.17、−0.36、−1.83，甚至 −9.7。高密度下 optDesign 的 upsize/buffer 會擠壞 placement 與繞線。
- **大範圍 upsizing ECO 會發散**：把整批違規 path 上的 cell 一起放大，setup 從約 −0.1 惡化到 −0.30／−0.67。原因是輸入電容變大，把上游的弱 driver 拖慢。
- **有效的作法是只換關鍵的 launch flop**：只把關鍵 path 起點的 DFFQ 換成 DFFH 類（clk→Q 較快），setup 收到 −0.022，hold、DRC、connectivity、antenna 與 post-sim 都乾淨。
- **剩下的幾十 ps 用手挑的 ECO 清單**：從 timing report 挑出單級特別慢的 XL gate 與 clk→Q 慢的 flop，逐顆執行 `ecoChangeCell -inst I -cell C`，接著 `refinePlace -eco true` 與 `ecoRoute`，再跑 `ecoRoute -fix_drc`、addFiller、檢查與輸出。每顆都 `catch` 並印出 rc，cell 名稱打錯才看得到。
- **換 netlist 要重跑 APR**：在同一個 floorplan 上換成另一版綜合的 netlist，結果是 −0.24 setup，post-sim 有上千個 timing violation。
- **Innovus 的 setup 與 post-sim 可以不一致**：setup −0.022 的 run，post-sim（native SDF）仍是 0 violation。但交付規則若要求 setup ≥ 0，post-sim 乾淨不能代替 setup。

相關：[[innovus-physical-check-points]]

## Restored DB: do not re-load or re-propagate constraints (verified 2026-11-24, Innovus 25.x, CVSD final SCE)
- On a `restoreDesign` of a post-route DB, both `update_constraint_mode -sdc_files ...` and a bare `set_propagated_clock [all_clocks]` wipe the I/O latency CCOpt set at CTS. Every in2reg/reg2out path then reads about -0.5 ns setup and hold (the `default` path group), while reg2reg stays correct.
- To retime under a changed uncertainty, change only that value: `set_interactive_constraint_modes [all_constraint_modes -active]; set_clock_uncertainty -hold 0.1 [get_clocks clk]; set_interactive_constraint_modes {}`.
- Curated single-path setup ECO (ecoChangeCell + refinePlace -eco + ecoRoute): upsizing a failing output flop to an HQ flop can move the miss to its D pin (larger setup). A same-family drive-up (DFFQX2 -> DFFQX4) plus upsizing the D-side high-fanout driver closed a -0.006 miss to 0.000.

## Route-DRC loop and run-to-run spread (verified 2026-11-25, Innovus 25.x, CVSD final SCE at 1.791 ns)
- A timing-gated DRC repair loop (`ecoRoute -fix_drc` + postRoute opt, up to 8 rounds) does not converge monotonically: later loops often trade DRC for -0.2 to -0.4 ns. Save a checkpoint (`saveDesign DBS/pr_best`) on every improvement, stop after 3 stalled loops, and restore the best before signoff. Rank loops as closed first, then hold ≥ 0, then WNS minus 5 ps per DRC marker; ranking "DRC 0 first" picked a -0.295 loop over a -0.071 / 2-DRC one.
- At a marginal period the same flow swings 0.1-0.3 ns run to run. A single good run (-0.004) did not reproduce on rerun (-0.06 to -0.30). Judge a lever by several runs, not one.
- Local M2/M3 short hotspots move with any placement perturbation. Shifting the M6 stripe start offset by 30 µm took one run from 114 to 10 DRC markers, but combining that with `place_global_cong_effort high` gave 45. Treat stripe offset, congestion effort and partial blockages as seed-like perturbations, not additive fixes.
- Single-cell upsizing ECOs next to SRAM macros fail `refinePlace -eco` legalization (IMPSP-2021), and deleting plus ECO-rerouting marker nets clears the DRC but costs about 60 ps. Leave whitespace near macros if post-route ECOs are planned.
- `reset_path_exception` on a restored DB needs interactive constraint mode, or it fails with TCLCMD-1048: `set_interactive_constraint_modes [all_constraint_modes -active]; reset_path_exception; set_interactive_constraint_modes {}`.
