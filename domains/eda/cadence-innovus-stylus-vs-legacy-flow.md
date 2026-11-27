---
title: Innovus 25.10：Stylus／Legacy 命令與 GUI 回呼的差異
scope: "domains/eda"
status: active
updated: 2026-11-04
---

## TCL、PDK 與畫面

- **舊教案命令在 Innovus 25.10 報 unknown。** 先核啟動模式 `innovus -legacy_ui`／`innovus -stylus`。Stylus 依序 `read_mmmc`、`read_physical -lef`、`read_netlist -top`、`init_design`；`floorPlan` 對應 `create_floorplan`、`addRing` 對應 `add_rings`、`sroute` 對應 `route_special`、`routeDesign` 對應 `route_design`。Legacy 的 verifyConnectivity／verify_drc、saveDesign／restoreDesign 在 Stylus 對應 check_connectivity／check_drc、write_db／read_db。CTS script 為 `clock_opt_design`，不要因 GUI 命令名稱相似就替代。下方原文保留完整參數對照。
- **UMC/Faraday 0.18 µm 的 PG 或 floorplan 無法接上。** FSA0M_A 是 VCC/GND、site `core_5040`、金屬 `metal1` 到 `metal6` 為小寫。先讀 `header6_V55_20ka_cic.lef`，再 core／antenna LEF。Stylus `create_floorplan -site core_5040 -core_density_size {aspect density margin1 margin2 margin3 margin4}` 保留 aspect/density/四邊 margin；PG 用 `connect_global_net` 並核 power pins、tie、core ring 和 special route。Lab5 不同技術的 placement grid 見 [CVSD Lab 陷阱](../../projects/ntu-cvsd-adfp/lab-execution-pitfalls.md)。
- **macOS X11 顯示 `IMPSYT-1507` 並掉到 no window mode。** 先用 XQuartz 確認本地顯示，檢查 SSH X11 forwarding 與 `XAuthLocation /opt/X11/bin/xauth` 是否用到目標 host；診斷不要先改所有 host 的 SSH 設定。教學的 RTL→網表→版圖可用 `get_db` 查暫存器的 `.base_cell.name`／`.location`，用 `select_obj` 高亮，再以 `report_timing` 讀實際違例。

## 25.10-p002_1 Qt GUI 驗證

- **GUI 表單欄位查到空值，或同名 action 執行別的事。** CUI 與 GUI legacy Tcl interpreter 分開，用 `eval_legacy {uiVar::Get RdaPG pins}` 讀實際欄位、`qtfForm::getactiveform` 驗 active form；menu 以完整 labels、唯一 ID、callback 核對。先開表單、讀回再觸發原生按鈕；`gui_replay_*` 是版本特定驗證介面，直接跑 APR 指令不能算原生操作證據。真實 menu 的 Add Ring 為單數，GUI form 為 Add Rings。
- **從 GUI 做出類似結果，被報成精確重現 script。** Place Standard Cell 執行 `place_design`，legacy Synthesize Clock Tree 是 `clockDesign`；嚴格重現走同一 graphical session 的 launch terminal，CTS 讀實際 report 及 CCOpt debugger。當時 Windows menu 沒有 Console／History／Log Viewer；`view_log <existing-log-file>` 開的是唯讀 viewer。
- **NanoRoute 的 Global Detail Route 和 Global＋Detail 看似同選項。** 同勾 Global/Detail 才會有 `routeDesign -globalDetail`，另名 Global Detail Route 是 `routeWithEco`。表單預設 End Iteration=1、Fix Antenna=true、Auto Stop=false，跟 script 的 auto／auto／true 不同；通過 DRC／connectivity／antenna 仍標 GUI variant。精確重現取消表單並從原 CTS checkpoint 的 fresh session 跑 `route_design`。SRoute 的 Follow Pins 對應 corePin，`noCorePins=1` 是開；Basic layer change 與 Via Generation 的 crossover／target range 分開核對。

[完整原文與實測紀錄](https://github.com/swear01/agent-memory/blob/93786b66aa982bda81d4c1ebe6cfceaa832b5d32/domains/eda/cadence-innovus-stylus-vs-legacy-flow.md)
