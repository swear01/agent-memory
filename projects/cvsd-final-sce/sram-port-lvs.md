---
title: "CVSD SCE SRAM 實體 port 與獨立 LVS"
scope: "projects/cvsd-final-sce"
status: active
updated: 2026-11-28
---

- SCE A2 和 tie 修正版的 merged GDS 在 CBDK IC Validator 都有頂層 VDD/VSS short，即使 tie 修正版 Innovus verifyConnectivity 無異常。A2 的同次 checkpoint 是 addFiller 後保存的 `outA2O_fill.dat`。從隔離副本執行 `saveNetlist -phys -excludeLeafCell -exportTopPGNets`，讓 CBDK 提供葉節點介面。ICV 2026.03 Nettran 預設生成 `$PINS`，會拒絕舊 wrapper 的 `-sp-dollarPins`。同批含 inline leaf 定義的 PG 網表曾在 ICV SPICE→NDB assertion 中止。
- TA 同批 1024x8 LEF 和 vendor GDS 的 116 個 signal rectangles 全差 `(+8,+8) µm`，PG strips 尺寸也不同。用正式 GDS port polygons 和 287.510×241.930 µm bbox 建 APR view，保持原始 GDS。只移 SREF 對齊訊號會讓原 LEF VDD 幾何打到 GDS VSS，必須重建 PG ports 後重新佈線。先用 GDS 做 macro port 與 spacing gate，streamOut 後再驗 raw port access、無黑箱 M4 和 CBDK LVS。
- CBDK Phantom frame 與本案 SRAM GDS 只有 pin/outline 抽象幾何。CBDK 的 `tsmc13_lvs.spi` 是空 subcircuit 介面。這組輸入可做獨立 interconnect 和 macro 邊界 LVS。元件級驗證需真實裝置 GDS 和相符 CDL。完整指令、正反對照、報告與 scripts 在 SCE repo `exp/area/lvs/`。

- SCE-16：如果 ICV 把 SRAM black-box pin 改名成 `ICV_<n>_<pin>`，先跑原名比較並保留萃取。以同一 GDS、含 PG 的 schematic 網表和 deck 重驗，只在空 subcircuit header 加入逐一核對的 pin alias。原名的 fixture LVS FAIL，套用 alias 後 24 顆、698 nets／ports 全配對，29 個 `text_open_rename` 仍存在。兩份萃取只差工具生成的 instance ID，依宏座標展開皆對到 696 個 signal 與 48 個 PG 頂層網。
- Innovus `verify_drc` 對 LEF obstruction 的間距和 CBDK 對 stream-out 的實體間距應各留報告。SCE-16 的 VIA45 cut 到 macro OBS 只剩 0.005 µm，但 vendor GDS 該層無 cut，ICV 對同次 GDS 0 違規。向外移 via 補 0.215 µm 反而縮小對 VSS 的實體間距，先核對 PG 再決定改 abstract 或重繞。證據在 SCE repo `exp/area/safe-fp/cj12-narrow-native-route.md`。

- SCE-21：Innovus 25.10 的 `saveDesign` 可在 `libs/lef/` 建指向原輸入的絕對 symlink。複製 checkpoint 後先查 `readlink`，替換副本時用原子 rename 解除 symlink。變更 LEF 觸發 `IMPIMEX-7024` 時，從鎖定 DEF 用新 LEF 重新匯入，保留 checkpoint 的完整性檢查。對固定 SRAM access，可依同次 stream 的實際 VIA45 cut 與 tech LEF 間距在 cut OBS 做局部窗口。SCE-21 用 29 個窗口把 VIA45 markers 由 356 降到 0。先比較完整 GDS 元素（只去日期與輸出順序）、PG 網表及 LVS SPICE，再以實體輸入等同性沿用 CBDK 結果。有新幾何時重驗正式 deck。[SCE-21 報告](https://github.com/swear01/cvsd-final-sce/blob/d6cac8920043d5be04f1710a0c160c6aed0fd5cd/exp/area/safe-fp/cj12-native-abstract.md)
