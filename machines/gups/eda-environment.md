---
title: GUPS workstation EDA environment (Innovus/Genus/Xcelium/Calibre, U18 PDK)
scope: machines/gups
machine: gups
status: active
updated: 2026-10-26
---

# GUPS 工作站（intl.gups.ntu.edu.tw）EDA 環境

Rocky Linux 8.10。`/usr/cad` 是 `/cad` 的 symlink，CIC 的 csh 腳本寫 `/usr/cad/...`，實體在 `/cad`。授權走 TSRI `lshc`（Cadence `5280@lshc`，Siemens `1717@lshc`）。

## 已安裝與入口

- Cadence DDI 25.10（Genus/Innovus 25.1）：`source /cad/cadence/CIC/innovus.sh`，`genus` 也在同一個 PATH（`/cad/cadence/DDI/cur/bin`）。
- Cadence Xcelium 25.03.005（2026-10-21 從 DVLab fleet `/apps/eda/cadence/XCELIUM/25.03.005` 以 tar 串流複製）：`/cad/cadence/XCELIUM/{25.03.005,cur}`，入口 `source /cad/cadence/CIC/xcelium.sh`（bash，PATH 加 `tools/bin/64bit`、`tools/bin`、`bin`）。`lshc` 有 `Xcelium_Single_Core` 240 套。以 `setpriv --reuid=nobody --regid=<PDK_approved gid>` 非 root 跑 `xrun` 驗證 UART／async FIFO 自檢 testbench 全過。
- Cadence Verisium 25.03.001（2026-10-22 同樣從 fleet 複製，檔案清單逐項一致）：`/cad/cadence/VERISIUM/{VERISIUM_25.03.001,cur}`；`xcelium.sh` 已設 `VERISIUM_DEBUG_ROOT`。`lshc` 有 `verisium_debug` 240 套。波形流程：`xrun -debug_opts verisium_pp -access +r ... -input probe.tcl -run -exit`（probe.tcl：`ida_probe -log -wave -wave_probe_args="tb -all -depth all"`）產生 `ida.db`，再 `verisium_debug -lwd xcelium.d -db ida.db &`。只有 `-debug_opts verisium_pp` 不會產生波形 DB。教學只用 Verisium，不教舊版 SimVision。
- 安裝前 gups 沒有任何 Verilog 模擬器（無 Xcelium、VCS、iverilog）；Synopsys 只有 HSPICE，Mentor 只有 Calibre 2025.3（`/cad/mentor/calibre/aok_cal_2025.3_28.17`）。`lshc` 另有 Questa/ModelSim `msim*` 授權但軟體未裝。
- 權限慣例：`/cad/cadence/DDI`、`/cad/cadence/XCELIUM`、`/cad/cadence/CIC/*.sh`、`<remote-home>/PDK` 為 `root:PDK_approved` 750。2026-10-21 時 `PDK_approved` 只有老師帳號 `ICD001`；2026-10-22 依使用者要求加入唯一的 wheel 管理員 `joinet`。前屆成員名單備份在 `/root/pdk_approved_members_backup_20260917.txt`。學生帳號建立後才加入群組。
- Innovus／Verisium／Calibre GUI 只走 X11（Innovus Qt 平台外掛只有 xcb、vnc、webgl、offscreen 等，無 wayland）；sshd 已開 `X11Forwarding`，伺服器有 `xauth`、`xdpyinfo`，沒有 `xclock`。

## U18（UMC 0.18um Faraday）重點

- CBDK：`<remote-home>/PDK/CBDK018_UMC_Faraday_v1.2`，只有一套 core library `FSA0M_A GENERIC_CORE`（382 cells）與 T33 IO；corner 為 `ss1p62v125c`／`tt1p8v25c`／`ff1p98vm40c`，無多 Vt、無多 track。`.lib` 預設 wireload `G5K`。
- `QDFZN` 等 `*DFZ*`／`DBZ*`／`JKZ*` 是 scan flop（`.lib` 有 `test_cell`）；沒做 DFT 時 Genus 會把 enable mux 吃進 scan flop，教學用 dont_use 清單要禁掉。`GCKET*` 是 ICG。
- 教學 UART（`/cad/tutorial/innovus_u18_tutorial`，2026-10-26 以非 root 帳號實測）：空 dont_use 時 Genus 用 2 個 `QDFZN`；20 ns 時脈下 post-route setup WNS 約 +13.9 ns（時脈很鬆）；ff corner hold 不修為 −0.205 ns、80 條違規；在 `clock_opt_design`、`route_design` 後各加 `opt_design -post_cts/-post_route -hold` 後 hold +0.001 ns、插入 38 個 `DEL*`；Calibre DRC 兩者皆 0。Cell family 說明以 `CIC/doc/FSA0M_A_GENERIC_CORE_DataBook_v1.0.pdf` 目錄為準，gups 有 `evince` 可經 X11 開。
- 同一 UART 的 CTS／floorplan 實測（2026-10-26）：不設限制時 `clock_opt_design` 插 0 個 clock buffer（`clk` port 直接驅動 76 個 flop，skew 4 ps），設 `set_db cts_max_fanout 16` 後為 5 個 `BUF*CK`、insertion delay ss 0.13 ns／ff 0.06 ns；`report_clock_trees -summary` 與 `report_skew_groups -summary` 都支援 `-out_file`。`create_floorplan -core_density_size` 的 utilization 拉到 0.85／0.95 時 hold 修不掉（−0.163／−0.199 ns），log 為 `Stopping hold fixing due to density exceeding max design density 95.000%`；0.3–0.7 皆可修到 hold ≥ 0。拿掉 `route_special` 時 step 04 的 `check_drc`、`check_power_vias` 仍全過，只有 step 07 `check_connectivity` 抓到 48 個未連接 terminal。
- Phantom GDS（`CIC/Phantom/fsa0m_a_generic_core_cic.gds`）只含 M1 與文字層，工作站上找不到真實 cell GDS；Calibre DRC 因此只驗得到 M1–M6/VIA，ANTENNA deck（需 PO1 AND DIFF）報 0 不代表通過。
- UMC 官方 Calibre DRC deck 不在 CBDK，而在老師下載的 FDK `UM180FDKMFC00000OA_B02.tar.gz` 的 `Calibre/DRC/`（BASE 2.16_P1 預設已開 `20KA_TOP_METAL`）。小區塊教學跑法要 `DRC UNSELECT CHECK METAL_DENSITY / DIFFUSION_DENSITY / RF_RULES`，否則只剩整晶片密度類違規。

相關：[[cadence-innovus-stylus-vs-legacy-flow]]、[[innovus-physical-check-points]]
