---
title: GUPS workstation EDA environment (Innovus/Genus/Xcelium/Calibre, U18 PDK)
scope: machines/gups
machine: gups
status: active
updated: 2026-10-27
---

# GUPS 工作站 EDA 環境

- GUPS 是 Rocky 8.10，CIC 腳本寫 `/usr/cad`，這台由 symlink 指到 `/cad`，與 DVLab fleet 的 NFS `/apps` 入口不同。Cadence/Siemens 依 TSRI `lshc` 授權，各工具須實際 checkout。Questa feature 存在不能推論軟體已裝。
- 新 Bash 終端明確 `source /cad/cadence/CIC/innovus.sh` 使用 DDI 25.10 的 Genus/Innovus，或 `source /cad/cadence/CIC/xcelium.sh` 使用 Xcelium 25.03.005 與 Verisium 25.03.001。子腳本設定的 PATH 無法留在父 shell，跑完模擬後 GUI 命令不存在先查是否在目前 shell source。Innovus/Verisium/Calibre GUI 走 X11，查 `X11Forwarding`、`xauth` 與 `xdpyinfo`，不要假設安裝了 xclock 或有 Qt Wayland 外掛。
- Verisium 用 `xrun -debug_opts verisium_pp -access +r ... -input probe.tcl -run -exit`，probe.tcl 須設定 `ida_probe -log -wave -wave_probe_args="tb -all -depth all"`，再 `verisium_debug -lwd xcelium.d -db ida.db`。單有 `-debug_opts verisium_pp` 不產生 waveform DB。這門課用 Verisium，舊 SimVision 不列進教材。
- 受限 EDA/PDK 樹用 `root:PDK_approved` 及 mode 750。核對實際群組資格，用非 root 帳號跑 smoke，不繼承上課世代的成員快照。`/cad/tutorial/innovus_u18_tutorial/` 與 tarball 為學生內容，答案留本機。舊 teacher instructions、`DONT_USE=` 和 `FIX_HOLD=1` 環境變數已退役。

## U18 教材容易踩的工具限制

- PDK 為 `<remote-home>/PDK/CBDK018_UMC_Faraday_v1.2`，只有 FSA0M_A GENERIC_CORE + T33 IO，無多 Vt／多 track。Wireload 預設 `G5K`。辨識 scan flop 以 liberty 的 `test_cell` 為準（例如 QDFZN、`*DFZ*`、`DBZ*`、`JKZ*`）；沒做 DFT 時 Genus 也可能以它吸收 enable mux，應加入教學 dont_use。`GCKET*` 是 ICG，cell family 查 vendor DataBook。
- UART 的寬鬆 setup 不證明 hold 沒問題。看 fast corner，測 `clock_opt_design` 和 `route_design` 後的 `opt_design -post_cts/-post_route -hold`。過密 floorplan 有 `Stopping hold fixing due to density exceeding max design density 95.000%`，代表實驗應保留足夠插 cell 空間。Density 由 site/row 量化，核對 `get_db designs .core_bbox` 和實際 core area。
- CTS 可以在小 design 用 port 直驅而插零 buffer，檢查 `report_clock_trees -summary`、`report_skew_groups -summary`。課堂測試可用 `set_db cts_max_fanout 16` 觀察 buffer 插入。限制 `design_top_routing_layer` 要在 CTS 前；routing 才限可能遇 `NRDB-2276`。該 UART 限 metal3 可跑完，metal2 在 Innovus 25.1 的 CTS 曾 `eGR Steiner is invalid`。
- 拿掉 `route_special` 時，power plan 的 `check_drc`、`check_power_vias` 仍可能過，routing step 要加 `check_connectivity` 才能發現未連端點。Route 後預設 multi-cut swap，要看 `report_route -multi_cut`；提高 detail multi-cut effort 不一定改結果，`set_db route_design_detail_post_route_swap_via none` 才能隔離 swap。在同 session 再 read DB 可能 `IMPIMEX-7031`，重開後從 checkpoint 續跑。
- 專案例用 `resume_from <上一步>` 讀 `ckpt/<step>.enc`。重跑實驗會覆寫 checkpoint，後續數據變成由實驗版算；重開 Innovus 並按預設重跑該步，或分離 lab checkpoints。缺 checkpoint 報 `IMPIMEX-7331`；init 後重 source init 的 read_mmmc 報 `IMPSE-110`；相對路徑錯誤先核對在 `run/` 目錄。
- Phantom GDS 只有 M1 和文字。Calibre 只測到相應 routing/via，需 PO1 AND DIFF 的 antenna 0 不代表真實 cell／製程 signoff。官方 deck 在老師下載的 FDK `UM180FDKMFC00000OA_B02.tar.gz` 的 `Calibre/DRC/`，不在 CBDK。小區塊教學若排除 METAL_DENSITY／DIFFUSION_DENSITY／RF_RULES，要明確標示其非全晶片檢查。
- 教學 UART RX 的 counter per bit 實際跑 N+1 cycle，50 MHz 到 5 Mbaud 的誤差可累積超過半 bit。TB stop bit 後只等 1 µs 的實驗會造成人為的不對稱，不能將結果當真實 baud 容忍度。

## 系統相容性

- `IC/cur` 是 IC 25.10；IC 6.1.8 在 Rocky 8 的 `libcrypto.so.10` 缺漏應先核對支援矩陣和真實使用需求。IC/DDI 25.10 自帶 support list 含 Rocky 8/9；6.1.8 自帶僅列 RHEL 6/7，不能把相容函式庫說成廠商支援。升系統先確認 kernel 對應 NVIDIA 模組，維護期間避免先升 userspace 造成 driver/library mismatch。重開分開驗 SSH、xrdp、GPU 和 GUI。
- AMD Threadripper 的 `mcelog.service` 曾報 unsupported CPU，服務已停用。OpenSSH 8.0p1 的 curve25519 KEX 可讓 10.1 以上 client 發 post-quantum 警告，應與連線失敗分開。

相關：[[cadence-innovus-stylus-vs-legacy-flow]]、[[innovus-physical-check-points]]。

[舊套件盤點、UART／CTS 逐次量測與維護收據](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/machines/gups/eda-environment.md)
