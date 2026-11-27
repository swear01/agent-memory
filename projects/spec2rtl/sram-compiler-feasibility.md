---
title: Spec2RTL SRAM compiler 可行性與 OpenRAM/OpenFinRAM 驗證邊界
scope: projects/spec2rtl
status: research-verified-integration-pending
updated: 2026-10-09
---

- **有 GDS/LEF／Liberty 就認定 SRAM 可 signoff。** OpenRAM 的 FreePDK45 三組實測是 512×8 1RW whole-word、512×45 1RW bit-mask、128×64 1R1W bit-mask，共 1,268 behavioral checks；未驗 same-address collision 或實際 Spec2RTL wrapper。原生環境需 `use_nix=False` 並先建 output directory。Analytical Liberty、可讀版圖、檔名 `.lvs.sp` 均不構成 characterization／DRC／LVS／PEX，TSRI 14-cell kit 也不是完整 compiler technology。
- **用 45nm/ASAP7 macro 替換原 canonical 40nm Conv。** 目前 grading 強制 canonical 512×8 behavioral SRAM 且拒絕 macro／blackbox。維持 golden；新實驗用同製程 standard cells、RC 與額外 target。Conv 原模型 async reset 全 array、posedge read、write hold Q；OpenRAM 是 posedge capture／negedge operation，Q 可能 X。要測初始化、時序及 wrapper。最新版 RLE 是 whole-word WEN 無 BWEB，仍要驗 45-bit 與 sleep pins，別搬舊 bit-mask blocker。
- **官方同名 `256×64` 拿去直接接 Mamba。** 要核實 `generated/verilog` 實際宣告，root 同名候選是 1024×64。保持同 depth／width／scale 的 LIB/LEF/Verilog；bank GDS 不等於生成 macro。官方 sample 是 single-port、whole-word，原 128×64 1R1W concurrent-read/write 和 masked-write 兩個反例均重現。換名或 padding 不會修 port／mask 行為。
- **OpenFinRAM 的 DFF mapping／pin-only DEF 檢查錯報。** 在原 Yosys 0.69+post 快照，診斷試驗的 `dfflibmap` 明確排除 `DFFHQx4_ASAP7_75t_R`、`DFFHQNx2_ASAP7_75t_R`、`DFFHQNx3_ASAP7_75t_R` 後才得到 fixture 要求的 mapping，保留 structural assertions 並重做 equivalence。DEF 用 `DISTANCE MICRONS`、`TRACKS X`；只有 pins、沒有 routed shapes 的 fixture 要讀 pin datatype 251（`40/251,50/251`），不是 routed metal 的 `40/0,50/0`。正式 patch 涵蓋 generator/tests/golden；這組診斷只判定 mapping 與 geometry，不能代替電性驗證。
- **OpenFinRAM 成功 exit 或 35-pin 幾何核對 PASS。** PR #8 修 DFF mapping/DEF fixture 及缺 SPICE 時拒絕 stub，14/14 CTest；`sram_x128x8x1` 的 128 是物理 WL 幾何，該宏邏輯為 512×8。Calibre 缺 tcsh 的那次跳過 LVS。補齊 CDL、在 Rocky8 用已授權 child wrapper 真實執行後為 **LVS INCORRECT**：log 顯示部分 data pins、VDD/VSS 黏同一 net，裸 bitcell 對照也不合。Pin 有 metal 不表示電性分離，沒有完整宏 DRC、PEX 或 SPICE characterisation。v2lvs 在容器 PID 1 的 license 假象與已實測修法見 [Calibre 授權筆記](../../domains/eda/calibre-container-licensing.md)。
- **已部署研究檔案就引用 OpenFinRAM 為 qualified。** 共用路徑 `/apps/cad/cell_library/ASAP7_EDK/memory-20261009/ram/`：OpenFinRAM 512×8 明列 `STATUS=LVS_FAILED`；攤平 SPICE 只去掉暫存依賴，沒有修電路。官方 views 是固定快照；OpenRAM FreePDK45 僅參考。OpenFinRAM 根目錄缺 LICENSE，README 標示不足確認再散佈；只附研究 repair patch。ROM 與驗證入口見 [ASAP7 subset](asap7-benchmark-feasibility.md)。

[完整原文與實測紀錄](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/projects/spec2rtl/sram-compiler-feasibility.md)
