---
title: Spec2RTL SRAM compiler 可行性與 OpenRAM/OpenFinRAM 驗證邊界
scope: projects/spec2rtl
status: research-verified-integration-pending
updated: 2026-10-08
---

# Spec2RTL SRAM compiler 可行性

研究追蹤：`DVLab-NTU/spec2rtl` issue #18。本輪研究完成；實際 integration/signoff 尚未完成。未變更共用 PDK、production CI、baseline 或 goldens。

## 固定來源

- OpenRAM stable：`b2b069ce119d1488cbe6883b2240bceb5c7ce29a`，BSD-3-Clause LICENSE。
- OpenFinRAM：`36be702767b2c45b842fbc800ee84c6dde4d34a4`，README 宣稱 BSD-3-Clause，但根目錄 LICENSE 缺失、GitHub license metadata 為 null；正式授權文字仍待釐清。
- Spec2RTL develop：`c8f82db743f1be8e5565ef8c8d36cc4990364ddb`；固定 CI benchmark `ad97739f1bad6ad78fb7ec5fe12da4ce437718c2`。較新 HW-Benchmark 需求查閱 `94bcd8e6af001019d433ac9f4d03049146061513`，不能当成固定 CI 已涵蓋。

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

OpenFinRAM 的「開源模式」表示 Yosys/OpenROAD/KLayout backend，另一路用 DC/Innovus；程式碼公開。開源路徑 single-port、偶數 width、缺現成 bit-mask，不可直接滿足 RLE 45-bit mask/Mamba 1R1W。estimated Liberty、read-path SPICE prototype、controller TT STA、instance/geometry LVS helper 都不等於完整 macro characterization/transistor LVS。

## Spec2RTL 整合限制

- `scripts/ci/grade.py` 使用 `CBDK_IC_Contest_v2.5/.../slow.db`，強制 canonical Conv SRAM、拒絕 macro/blackbox；其 PPA 是 SRAM behavioral model 的 standard-cell mapping。需要額外 experimental target，不可偷換 baseline。
- 45nm/ASAP7 macro 必須配同製程 standard cells 與 RC；不能跨製程混用後宣稱 PPA 可比。
- Conv canonical 512×8 model asynchronous reset 整個 array、posedge read、write 保持 Q；OpenRAM 沒有相同 reset/init 契約，posedge capture/negedge operation，Q 會變 X。需驗證 wrapper/init/latency，而不是改 golden 或靜默 FF fallback。
- RLE mask 極性、sleep pins；Mamba 同址 collision 與寬記憶體 partitioning 仍需設計和驗證。

證據：研究 workspace 的 `outputs/sram-compiler-research-20261008/`，Mazu `/var/tmp/sram-compiler-research-20261008/` 保留 config、logs、benches、views audit/hash manifest 和 macro 產物。後者是暫存資料，尚未部署 NAS。Issue #18 comment 保存配置、結論與驗證範圍。
