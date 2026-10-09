---
title: Spec2RTL SRAM compiler 可行性與 OpenRAM/OpenFinRAM 驗證邊界
scope: projects/spec2rtl
status: research-verified-integration-pending
updated: 2026-10-09
---

# Spec2RTL SRAM compiler 可行性

研究追蹤：`DVLab-NTU/spec2rtl` issue #18。本輪研究完成；實際 integration/signoff 尚未完成。未變更共用 PDK、production CI、baseline 或 goldens。

## 固定來源

- OpenRAM stable：`b2b069ce119d1488cbe6883b2240bceb5c7ce29a`，BSD-3-Clause LICENSE。
- OpenFinRAM：`36be702767b2c45b842fbc800ee84c6dde4d34a4`，README 宣稱 BSD-3-Clause，但根目錄 LICENSE 缺失、GitHub license metadata 為 null；正式授權文字仍待釐清。
- Spec2RTL develop：`c8f82db743f1be8e5565ef8c8d36cc4990364ddb`；固定 CI benchmark `ad97739f1bad6ad78fb7ec5fe12da4ce437718c2`。初輪較新 HW-Benchmark 需求查閱 `94bcd8e6af001019d433ac9f4d03049146061513`，不能當成固定 CI 已涵蓋。2026-10-09 最新 `ed2c429f5cd8a36876081fb0b0b9769935994c3f` 的七個 40nm designs、RLE 介面變更及 ASAP7 subset 實測見 `asap7-benchmark-feasibility.md`。

## 實測與採用方向

Mazu 隔離研究，不修改全域 packages。OpenRAM 使用 upstream `technology/freepdk45`；TSRI 的 14-cell digital kit 不是完整 OpenRAM technology。Python 3.12 venv、KLayout 0.30.12、Icarus 12.0；OpenRAM 預設 use_nix=True，native 執行需明確設 `use_nix=False` 並建立 output directory。

| OpenRAM 配置 | 生成耗時 | 獨立 Verilog checks |
|---|---:|---:|
| Conv 512×8 1RW，write_size=8 | 54.6 s | 513 passed |
| RLE 512×45 1RW，write_size=1 | 365.1 s | 559 passed |
| Mamba 128×64 1R1W，write_size=1 | 212.7 s | 196 passed |

RLE/Mamba 首輪 150 s timeout，450 s 重試成功。三組均取得 GDS/LEF/Verilog/SPICE/analytical Liberty；KLayout 可讀、唯一 top/非空 geometry，LEF 與 Verilog 全 pins 一致。測試涵蓋全地址、chip deselect、bit mask/zero-mask，Mamba 不同地址同時讀寫。總計 1,268 behavioral-model checks；未測 same-address collision 或 project wrapper。

配置 `analytical_delay=True, check_lvsdrc=False, use_pex=False`；1.0 V/25 C，輸出 TT/FF/SS analytical Liberty。多 port analytical 模式各 port 使用第一個 read port timing。`.lvs.sp` 存在不表示 LVS 執行；未跑 DRC/LVS/PEX、SPICE characterization 或完整 Spec2RTL regression。優先候選是 OpenRAM/FreePDK45，並非已可投入 signoff。

OpenFinRAM Release build 成功；補齊依賴後 upstream CTest 11 passed/2 failed/0 skipped。Yosys 0.69+post 把 DFF map 成 `DFFHQx4_ASAP7_75t_R`，hard-coded 檢查要求 `DFFHQNx1_ASAP7_75t_R`，equivalence 六組與實際生成皆在此失敗。這是 mapping 名稱假設失配，未證明硬體功能錯誤。DEF→GDS fixture 含非法 `UNITS DISTANCEMICRONS`，KLayout parse 失敗。未修改 source 或刪除檢查；尚缺 OpenROAD executable，無完整 OpenFinRAM macro views。

同日後續隔離診斷副本確認小修路線：`dfflibmap` 加上 `-dont_use DFFHQx4_ASAP7_75t_R -dont_use DFFHQNx2_ASAP7_75t_R -dont_use DFFHQNx3_ASAP7_75t_R`，保留 structural assertions，六組 formal equivalence 全部通過。正式修補需同步 production generator/tests/golden；尚未實作或驗證 production generation。

DEF fixture 改 `DISTANCE MICRONS`、`TRACKS X` 後，還需修 test 的 layer purpose：fixture 只有 pins、無 routed segments，converter 的 pins_datatype=251，test 錯查 `40/0,50/0`。改查 `40/251,50/251` 後 smoke test 通過（2 cells、2 instances、非空 pins）；保留 DBU/outline warnings，不能推論實際 routed-controller connectivity 已驗證。以上診斷不改原 CTest baseline，port/mask 限制仍未解決。

OpenFinRAM 的「開源模式」表示 Yosys/OpenROAD/KLayout backend，另一路用 DC/Innovus；程式碼公開。開源路徑 single-port、偶數 width、缺現成 bit-mask，不可直接滿足初輪 RLE 45-bit mask/Mamba 1R1W。注意最新版 HW-Benchmark RLE 已換成 whole-word WEN，無 BWEB；45-bit 寬度與 macro 時序仍需驗證，詳見 `asap7-benchmark-feasibility.md`。estimated Liberty、read-path SPICE prototype、controller TT STA、instance/geometry LVS helper 都不等於完整 macro characterization/transistor LVS。

## Spec2RTL 整合限制

### OpenFinRAM 正式 patch 與目前 blocker

同日已建立 upstream OpenFinRAM PR #8（ready-for-review），fork branch `fix/open-flow-dff-def-20261008`，commit `d7e66ff0b20e48498a9c8295ba39a0d8c533185f`，remote SHA 讀回一致；upstream 尚未審核/合併。修 production/formal/golden DFF mapping、合法且含 routed segments 的 DEF fixture、保留 control/address pin 的原 M4/M5 layer；移除 `v2lvs` missing/failure 的 stub/partial-netlist 假成功，拒絕成功但缺 output。

Release build 與 14/14 CTest 通過；新增 SPICE failure regression 在 unchanged upstream 兩案例均失敗。KLayout pin audit 在舊產物抓到 16 個 label 缺同層金屬，修正後 35 GDS/LEF pins 都有金屬、0 LEF fallback rectangles。此診斷 GDS 含 4,096 個 `sram_cell_6t_122` bitcells，8-bit data/9-bit address 對應 512×8；`sram_x128x8x1` 名稱的 128 是 physical WL geometry，不能當 logical depth。

固定官方 `openroad/orfs` image digest `sha256:b879915e0ec547a7111e4e765f95b8896b3876d2ee36cf48d139355252678711`，controller placement/CTS/routing/TT global-parasitic STA 完成：108 delay INVx1、128 WL buffers、100 constrained max paths、controller route DRC report 空白。不等於整個 SRAM macro signoff。

2026-10-08 的 generation blocker：Calibre `v2lvs` 2026.3_27.19 在 Ubuntu 26 不支援，研究 Rocky 8 wrapper 回報授權取得失敗；當時尚未定位根因。最終 patch 正確 exit 1，沒有 stub 當 macro。當日 35-pin／容量 GDS 是套用 SPICE failure guard 前的診斷產物，不能部署 NAS。

2026-10-09 根因已實測定位於研究 wrapper 讓 v2lvs 成為容器 PID 1；只移除內層 `exec`，相同設定便能取得真實 `calibrelvs` 授權，兩組 A/B 重現失敗／成功。controller SPICE 129,625 bytes、2,805 X instances，與既有 docker-shell 轉換雜湊相同。未變更共用授權環境或 TSRI IP；不是缺少 Siemens 授權。細節見 `domains/eda/calibre-container-licensing.md`。

修後原 512×8 配置生成 exit 0：結果 `sram_x128x8x1_20261009_023319` 的 GDS 1,128,400 bytes、LEF 5,706 bytes、SPICE 5,232 bytes、estimated Liberty 10,035 bytes；35-pin audit 通過、0 fallback rectangles。但 LVS 因缺 tcsh 被 main 警告後跳過；exit 0 不代表 LVS passed。完整 DRC/LVS/PEX/characterization/Spec2RTL regression、netlist connectivity、port/mask 未完成，未部署 NAS。

- `scripts/ci/grade.py` 使用 `CBDK_IC_Contest_v2.5/.../slow.db`，強制 canonical Conv SRAM、拒絕 macro/blackbox；其 PPA 是 SRAM behavioral model 的 standard-cell mapping。需要額外 experimental target，不可偷換 baseline。
- 45nm/ASAP7 macro 必須配同製程 standard cells 與 RC；不能跨製程混用後宣稱 PPA 可比。
- Conv canonical 512×8 model asynchronous reset 整個 array、posedge read、write 保持 Q；OpenRAM 沒有相同 reset/init 契約，posedge capture/negedge operation，Q 會變 X。需驗證 wrapper/init/latency，而不是改 golden 或靜默 FF fallback。
- 初輪 RLE mask 極性、sleep pins；Mamba 同址 collision 與寬記憶體 partitioning 仍需設計和驗證。最新版 RLE whole-word write 契約另見 ASAP7 benchmark note，不可沿用舊 mask blocker。

證據：研究 workspace 的 `outputs/sram-compiler-research-20261008/`，Mazu `/var/tmp/sram-compiler-research-20261008/` 保留 config、logs、benches、views audit/hash manifest 和 macro 產物。後者是暫存資料，尚未部署 NAS。Issue #18 comment 保存配置、結論與驗證範圍。

## 完整 Calibre LVS 續查與拒絕 qualification（2026-10-09）

不再停在缺 tcsh：隔離研究用等價 bash 直接跑已授權 Rocky8 Calibre，保留 child-process wrapper；先校正 build/src paths，補齊 CDL，排除沒有 terminals/devices 的 physical-only FILLER/TAPCELL instances，沒有刪除有 pins 的 logic/data instances。

實際完整 compare 結果 **LVS INCORRECT**，提供的 ASAP7 deck/layout/source：ports32/35，nets10538/11297，initial transistors33337/28580；抽取 log 顯示 D[0]、D[4]、vdd、vss 為同一 net。裸 bitcell 對照亦不正確（8/5 ports），因此 source/layout/deck 相容性與供電 connectivity 校準仍未完成。這不是 foundry certification；但足以拒絕目前 macro 作為 qualified SRAM。先前35-pin同層金屬audit只證明label/geometry存在，不保證electrical separation或LVS。未跑完整macroDRC/PEX/SPICE characterization，未部署NAS。

官方 `The-OpenROAD-Project/asap7_sram_0p0` 有現成views；抽查256×64是 single-port、whole-word write。隔離bench對原Mamba128×64 1R1W wrapper重現兩個反例：同時讀addr0/寫addr1回AAAA而candidate保持BBBB；masked寫5回AAA5而candidate覆寫0005。不要把rename/padding當成multi-port/mask integration；需要合適同製程SRAM或修改並驗證存取protocol。

完整MambaASAP7synthesis/白箱gateharness與SDF校準現況接續 `asap7-benchmark-feasibility.md`。完整LVScomparison及failed/retry evidence保留於 `<research-root>/outputs/asap7-default-readiness-20261009/`，production CI/baseline/goldens/shared PDK皆未修改。

## 共用保存位置（2026-10-09 後續部署）

研究產物已依使用者要求保存在 `/apps/cad/cell_library/ASAP7_EDK/memory-20261009/ram/`：`openfinram-experimental/`有512×8 GDS/LEF/estimatedLiberty與攤平本次failed-LVS source的完整SPICE，保留SOURCE-HASHES與STATUS=LVS_FAILED；只是移除暫存includes依賴，未修connectivity。compiler僅附repair.patch，原source缺LICENSE仍待釐清。

`official-asap7-sram/`為官方固定 `9f5af0939e8dd3cc1a9693a50b23441691dd7d25` 的完整source/views。256×64 probe用generated/verilog，root verilog同名module容量不同不可混用；interface-counterexamples重現兩個反例。`openram-freepdk45-reference/`保存三組45nm views/config/LICENSE，不能充作ASAP7 macro。整合README與部署/重跑證據見 `asap7-benchmark-feasibility.md` 最後一節。上述舊段落「未部署NAS」描述部署前研究階段；現為共用研究保存，RAM資格仍未通過。
