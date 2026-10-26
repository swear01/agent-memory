---
title: GUPS workstation EDA environment (Innovus/Genus/Xcelium/Calibre, U18 PDK)
scope: machines/gups
machine: gups
status: active
updated: 2026-10-27
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
- 同一 UART 的 CTS／floorplan 實測（2026-10-26）：不設限制時 `clock_opt_design` 插 0 個 clock buffer（`clk` port 直接驅動 76 個 flop，skew 4 ps），設 `set_db cts_max_fanout 16` 後為 5 個 `BUF*CK`、insertion delay ss 0.13 ns／ff 0.06 ns；`report_clock_trees -summary` 與 `report_skew_groups -summary` 都支援 `-out_file`。`create_floorplan -core_density_size` 的 utilization 拉到 0.85／0.95 時 hold 修不掉（−0.163／−0.199 ns），log 為 `Stopping hold fixing due to density exceeding max design density 95.000%`；0.3–0.7 皆可修到 hold ≥ 0。同一 session 內可直接重下 `create_floorplan`（只出 `IMPFP-325` resize 警告），再 source floorplan step 的腳本即回到原 floorplan；core 會對齊 site 0.62 µm × row 5.04 µm，所以 cell area ÷ core area 略低於設定值（0.5→0.499、0.85→0.847）；`get_db designs .core_bbox` 讀 core 框。拿掉 `route_special` 時 power plan step 的 `check_drc`、`check_power_vias` 仍全過，只有 routing step 的 `check_connectivity` 抓到 48 個未連接 terminal。`set_db design_top_routing_layer` 必須在 CTS 前（例如 `init_design` 後）設定，只在 routing step 才設會因 CTS clock net 的 preferred layer（metal3–4）報 `NRDB-2276` 而不 route；設 metal3 全流程無差異（DRC 0、timing 幾乎不變），設 metal2 則 Innovus 25.1 在 CTS 階段 `eGR Steiner is invalid` 崩潰。
- Innovus 25.1 `route_design` 預設會在 route 後把 via 換成 multi-cut（log：`auto set of droutePostRouteSwapVia to multiCut`），UART 上 91.7% 為 2-cut；`set_db route_design_detail_use_multi_cut_via_effort high` 幾乎無差，`set_db route_design_detail_post_route_swap_via none` 則 0% multi-cut、DRC 仍 0。`report_route -multi_cut` 列 single/multi-cut 統計。同一 session 第二次 `read_db` 會被 `IMPIMEX-7031` 拒絕（需 `read_db_stop_at_design_in_memory false`），教學上改用重開 Innovus 由 checkpoint 續跑。
- Phantom GDS（`CIC/Phantom/fsa0m_a_generic_core_cic.gds`）只含 M1 與文字層，工作站上找不到真實 cell GDS；Calibre DRC 因此只驗得到 M1–M6/VIA，ANTENNA deck（需 PO1 AND DIFF）報 0 不代表通過。
- UMC 官方 Calibre DRC deck 不在 CBDK，而在老師下載的 FDK `UM180FDKMFC00000OA_B02.tar.gz` 的 `Calibre/DRC/`（BASE 2.16_P1 預設已開 `20KA_TOP_METAL`）。小區塊教學跑法要 `DRC UNSELECT CHECK METAL_DENSITY / DIFFUSION_DENSITY / RF_RULES`，否則只剩整晶片密度類違規。
- 教學 UART RTL 模擬（2026-10-26 實測）：`tb/tb_rx.v` 的 `BIT_RATE` 在 50 MHz 時脈下 115200／921600／3000000（`CYCLES/BIT` 434／54／16）全過，5000000（10）`FAILS 10`。原因是 `uart_rx.v` 的 `cycle_counter` 數 0 到 `CYCLES_PER_BIT` 才由 `next_bit` 歸零，每 bit 實際 N+1 cycle，誤差隨 bit 累積超過半 bit。改 tb 送端速度 ±x% 的結果嚴重不對稱，主因是 tb 在 stop bit 後只等 1 µs 就檢查，不是 UART 真實容忍度，不適合出題。
- 教材每個 step 用 `resume_from <上一步>` 讀 `ckpt/<step>.enc`，所以練習若重跑某 step（例如關掉 multi-cut swap 再跑 08_route）會覆寫該 checkpoint，後面 step 09 的 timing 就改從實驗版算（setup WNS 14.109 vs 預設 14.059）。2026-10-26 改成練習結尾要求重開 Innovus 用預設再跑一次該 step。
- 2026-10-27 起 `/cad/tutorial` 只放學生包 `innovus_u18_tutorial/` 與其 tarball；answer key 只在本機、不部署。10/22 的舊 `innovus_u18_tutorial_teacher/TEACHER.md`（Lab A/B/C、`DONT_USE=`/`FIX_HOLD=1` 環境變數，已不適用）已由使用者刪除。
- 新登入的 shell（`env -i bash -l`，2026-10-26 以 nobody 實測）PATH 裡沒有 `xrun`、`verisium_debug`、`genus`、`innovus`；每個新終端都要先 `source /cad/cadence/CIC/innovus.sh`（Genus/Innovus）或 `source /cad/cadence/CIC/xcelium.sh`（Xcelium/Verisium）。腳本內部 source 的環境不會留在互動 shell，所以 `run_sim.sh` 跑完後直接下 `verisium_debug` 會 command not found。
- 教材常見 Innovus 錯誤（同日實測）：`resume_from` 讀不存在的 checkpoint → `IMPIMEX-7331 Cannot identify the top cell name`；同一 session 再 source 一次 init step → `read_mmmc` 在 `init_design` 後重跑，報 `IMPSE-110`（`TCLCMD-1306`）；不在 `run/` 下 `source ../scripts/...` → `IMPSE-122 ... could not be located`。

相關：[[cadence-innovus-stylus-vs-legacy-flow]]、[[innovus-physical-check-points]]
