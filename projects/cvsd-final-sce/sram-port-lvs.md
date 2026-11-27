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
- SCE-18 的 PG 萃取 `SCE.net.gz` 將相鄰 SRAM 重包為合成階層 `VCELLR1`。直接數頂層 leaf 只見 4 顆，逐層沿 formal mapping 展開 12 個雙宏子電路後是 28 顆、56 個 PG terminals。同一份 GDS 在 cthulhu Rocky 中使用 `exec icv` 停在 license 啟動，保留 Bash 並由子行程執行 ICV 後正式 deck 完成。
- SCE-18 在 cthulhu 用專案的 `source "$HOME/cvsd_final_apr/invenv.sh"` 啟動 Innovus 25.10、checkout 八核並完成 placement。直接 source 公用入口缺 amd64 `libXp.so.6`，只補 libXp 又缺 `libtatlas.so.3`。既有專案腳本載入 CIC wrapper、使用者相容庫、工具所附 RHEL9 相容庫和 Ubuntu 系統 libstdc++ preload。
- SCE-25：相同 native SRAM pin 的多層接觸可在 ICV 成為不同頂層根。先用實際接觸層、座標與精確 DEF／physical 網表核對兩根的完整 driver/load，再決定是否接線。若 landing 附近的 PG rail 擋住短疊孔，可在宏外以實際金屬和 via 接回兩根。用原名 CBDK 重萃後核對目標實端聯集、所有其他萃取端點集合與 PG 集合，並重驗 DRC。即使 `text_open_rename` 診斷仍在，實際外部接線可證明單網修復。缺同版 SRAM device GDS／CDL 時，宏內連通仍未知。證據見 SCE repo `exp/area/safe-fp/sce25-native-net.md`。

- LVS 的 black-box comparator 可能略過只接未配對 macro ports 的萃取根。即使成對報出的根聯集涵蓋預期邏輯端點，同一 instance／pin 的其他 landing 仍可能落在額外根。以同次 landing 表逐層核對 formal，再沿原萃取階層展開所有接觸根。逐網核對聯集、外網端點與獨立 PG roots，並清點每個跨層接點牽涉的根，再判定單次接線是否涵蓋完整分網。
- SCE-28：Innovus 25.10 可用 `defIn -nets -incremental_routing` 匯入只含一個 signal net 的 DEF，保留既有 regular routes，再另存並 fresh restore 驗證。若鎖定的原 GDS 在首次 restore 之前匯出，先用相同 stream map 做一次未改 DB 的 restore/stream 對照。SCE 的純對照也把 VDD/VSS 頂層文字從 M6 移到 M7，且座標和 ECO stream 相同。以狀態與內容比對 DEF SPECIALNETS、用幾何內容比對自動命名 via，再驗全層幾何差集和 CBDK 萃取。這能判斷差異來自標籤選點或實際外網變更。逐 GDS 元素零差異的門檻仍需計入標籤。 [SCE-28 報告](https://github.com/swear01/cvsd-final-sce/blob/60c4af02070b5b884b61a6e8ecdc9c82617027aa/exp/area/safe-fp/sce28-native-route-db.md)

- SCE-32：遇到 CBDK Phantom 標準元件的 leaf formal 少一腳，先核對原始 GDS 同名 pin 金屬與完整 label，再用單顆原廠 leaf 做同一 deck 的 A/B，只在頂層 GDS 與 physical Verilog 同步引出目標腳。TLATXL 的 QN 在兩組原廠 leaf 都存在，未引出時萃取五腳且原廠 black-box 只缺 QN，同名引出後萃取六腳並配對。依萃取 leaf formal 順序核對每個 instance 端點、其餘腳同名且各自獨立，才能將差異歸因至目標腳表示。適用範圍為 Phantom 介面。[SCE-32 報告](https://github.com/swear01/cvsd-final-sce/blob/c62794721f1a8eace0026b37b23e77cdc452fa9c/exp/area/a2-clean-repair/sce32-tlatxl-leaf-fixture.md)

- 移除 SRAM 後先重建 PG 根。宏的 PG strip 本身可串接多層金屬，移除 SREF 後即使剩下的 macro PG terminals 全通，局部 via 支路仍可能浮接。依原網名與新根找出沒有保留端點的浮接支路，再逐座標重數同網雙 rail 與 placement keepout。裁 die 時以完整 site 框判斷，微量邊裁便可能少掉整列的邊側 sites。保留端點、站點與裁後金屬都用同一份新 GDS／DEF 重讀，異動過的 GDS 另跑適用 deck。 [SCE-36 裁框證據](https://github.com/swear01/cvsd-final-sce/blob/41bd4a6e0b9e4c3da7bb72cf989d05777c9a5768/exp/area/safe-fp/sce36-23sram-legal-die.md)
- SCE-38：同網 bridge 的異網 spacing 檢查通過後，仍要在加上 via enclosure 的金屬聯集上量凹口。原 M1 水平支路下緣距同網 VIA12 上緣只有 0.120 µm，CBDK `M1.S.1` 要 0.180 µm，雖然萃取已把 5 根收成 1 根且其他網均相同。離線看 merged polygon 的相向邊，正式判定沿用同一 fresh-restore control 與 CBDK deck。 同一未改 checkpoint 向外移 via 65 nm 到 5 nm grid，先以舊 notch 當反例驗證聯集檢查，再在實際 fresh-restore stream 量得 0.185 µm，原新增違例消失且同網萃取相同。 [SCE-38 報告](https://github.com/swear01/cvsd-final-sce/blob/d872b7857612b064fe999a4a27935c6757acd3c5/exp/area/safe-fp/sce38-a5-native-bridges.md)
- SCE-40：原名 LVS comparator 列的數字可能和原始 `SCE.net.gz` 的根不同。SCE-38R 中 `59897` 在 comparator 的 A[1] 輸出出現於另一網，沿同次 GDS 落點與 formal bindings 可確認其實接 C06 的 DIODE_2/A。先按 driver、diode 和各層 SRAM landing 重建每組 raw endpoint partition，再組合通過 spacing／notch 的局部 DEF。這樣一次匯入三條網後，CBDK opens 3→0、所有非目標 partitions 原樣，305 個原名 black-box port errors 的 master 與座標仍相同。 [SCE-40 報告](https://github.com/swear01/cvsd-final-sce/blob/07d426706bbdf693b56d5fc006f496ef774471ce/exp/area/safe-fp/sce40-remaining-sram-opens.md)
