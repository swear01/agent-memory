---
title: "Cadence Innovus 25.10 Stylus Common UI 與 Legacy UI 實作與 UMC 0.18µm 整合規範"
scope: "domains/eda"
status: active
updated: 2026-10-21
---

# Cadence Innovus 25.10 Stylus Common UI 與 Legacy UI 實作與 UMC 0.18µm 整合規範

## 背景與版本定位

1. **版本命名與釋出週期**：
   - Cadence EDA 工具採用「年份.版本」格式（例如 20.10、21.10、24.10、**25.10**）。
   - TSRI 提供的 `DDI_25.10.000`（Innovus v25.10-p002_1，建置於 2025 年 4 月）為 2025–2026 學年度最新的正式發布版本。
   - 工具目錄通常以軟連結維護（如 `<cad-root>/cadence/DDI/cur -> DDI_25.10.000`），更新新版或 ISR 時可透過抽換軟連結達成無縫升級。

2. **Legacy UI vs. Stylus Common UI**：
   - **Legacy UI**：相容 20 年前 First Encounter 指令架構（駝峰式如 `floorPlan`, `addRing`, `sroute`, `placeDesign`, `routeDesign`）。IC 競賽（教育部 IC Contest）考古題與臺灣多數高校舊講義仍以此為主。
   - **Stylus Common UI**：Cadence 現代統一架構，與前端合成（Genus）、靜態時序簽核（Tempus）及電源分析（Voltus）共用同一套指令結構與資料庫存取模型（底線小寫 snake_case、物件導向 `get_db` / `set_db`）。
   - Innovus 25.10 啟動時已發布官方 Deprecation 宣告：未來大版本將預設為 Stylus Common UI，Legacy UI 即將退役。

---

## 關鍵流程指令對照表

| 階段 | Legacy UI 指令 | Stylus Common UI 指令 | 注意事項與細節 |
| :--- | :--- | :--- | :--- |
| **啟動模式** | `innovus -legacy_ui` | `innovus -stylus` | Stylus 圖形介面採用 Qt5，支援深淺色主題與現代儀表板。 |
| **MMMC 設定** | `init_design -setup ... -hold ...` | `read_mmmc <mmmc_file>` | Stylus 中 MMMC 建議獨立為專用檔案，透過 `read_mmmc` 讀入。 |
| **實體庫與網表** | `set init_lef_file ...; init_design` | `read_physical -lef <lef_list>`<br>`read_netlist -top <top> <v_file>`<br>`init_design` | 實體與邏輯分開讀取，最後呼叫 `init_design`。 |
| **全域電源** | `globalNetConnect VCC -type pgpin ...` | `connect_global_net VCC -type pg_pin ...` | 包含 `pg_pin`、`tie_hi`、`tie_lo`。 |
| **Floorplan** | `floorPlan -r 1.0 0.7 15 15 15 15` | `create_floorplan -site core_5040 -core_density_size {1.0 0.7 15 15 15 15}` | Stylus 的 `-core_density_size` 參數需傳入陣列包含 aspect_ratio、density 與四邊 margin。 |
| **Power Ring** | `addRing -type core_rings ...` | `add_rings -type core_rings ...` | 鋪設外圍 VCC/GND 核心電源環。 |
| **Special Route** | `sroute -connect {blockPin corePin ...}` | `route_special -connect [list core_pin] ...` | 連結標準單元列內部電源軌至 Core Ring。 |
| **Placement** | `setPlaceMode ...; place_opt_design` | `set_db place_opt_run_global_place full`<br>`place_opt_design` | 擺放邏輯閘並進行 Pre-CTS 延遲最佳化。 |
| **CTS** | `clock_opt_design` | `clock_opt_design` | 時脈樹合成與 Skew 平衡。 |
| **NanoRoute** | `setNanoRouteMode ...; routeDesign` | `route_design` | 詳細金屬層繞線。 |
| **Filler 單元** | `addFiller -cell {...}` | `set_db add_fillers_cells {...}`<br>`add_fillers` | 填補空白列以符合 N-well 與微影連續性。 |
| **實體驗證** | `verifyConnectivity`<br>`verify_drc` | `check_connectivity`<br>`check_drc` | 檢查開短路與幾何 DRC。 |
| **資料庫儲存** | `saveDesign <name>.enc` | `write_db <name>.enc` | 儲存版圖狀態。 |
| **資料庫載入** | `restoreDesign <name>.enc.dat <top>` | `read_db <name>.enc` | Stylus 中以 `read_db` 載入資料庫。 |

---

## Faraday UMC 0.18µm（FSA0M_A）整合注意事項

1. **電源端點命名**：
   - Faraday FSA0M_A 標準單元庫嚴格採用 **`VCC`** 與 **`GND`**（非 TSMC 慣用的 VDD/VSS）。
2. **Site 名稱**：
   - Tech LEF（`header6_V55_20ka_cic.lef`）內定義的 Core Site 名稱為 **`core_5040`**。
3. **LEF 載入順序**：
   - 第一個必須是 Tech LEF（`header6_V55_20ka_cic.lef`），後續接 Core LEF（`fsa0m_a_generic_core.lef`）與 Antenna LEF（`FSA0M_A_GENERIC_CORE_ANT_V55.lef`）。
4. **金屬層名稱**：
   - 必須全小寫：`metal1` ~ `metal6`。
5. **Filler 單元名稱**：
   - `FILLER1` ~ `FILLER64`（包含 `FILLERBC`, `FILLERAC`, `FILLER8C` 等）。

---

## macOS XQuartz 與 X11 Forwarding 設定要點

在 macOS Terminal 透過 SSH 轉發 X11 執行 Innovus GUI 時，若遇到 `IMPSYT-1507: The display is invalid and will start in no window mode`：
1. **原因**：macOS Terminal 預設未設定 `$DISPLAY` 環境變數，且 `/usr/bin/ssh` 未指定 `XAuthLocation`。
2. **解法**：
   - 本地 `~/.ssh/config` 需加入：
     ```sshconfig
     Host *
       ForwardX11 yes
       ForwardX11Trusted yes
       XAuthLocation /opt/X11/bin/xauth
       ForwardX11Timeout 0
     ```
   - 在啟動 SSH 前確保本地環境變數帶入：
     ```bash
     [ -z "$DISPLAY" ] && export DISPLAY=:0
     ```
   - XQuartz 偏好設定中需確認 `enable_iglx=true` 與 `nolisten_tcp=false`。
