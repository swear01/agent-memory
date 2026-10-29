---
title: 大型開源 RTL 轉成組合 AIG：sv2v → yosys → unflop → ABC comb，與踩過的坑
scope: tools/yosys
tool: yosys
project: ntu-lsv-course
status: active
created: 2026-10-30
updated: 2026-10-30
tags: [yosys, abc, sv2v, aig, blif, nvdla, blackparrot, vortex, CULS, benchmark]
---

# 大型 RTL → 組合 AIG

目的：給 CULS-fast 做「真實設計」樣本。實測三個設計都成功（yosys 0.69+post `143eb14f`、ABC 1.01、sv2v v0.0.13，全在 mazu CPU 上）：

| 設計 | AND | level | latch | yosys 耗時 / 峰值記憶體 |
|---|---|---|---|---|
| NVDLA nv_full（nvdla/hw `8e06b1b9`） | 15,800,863 | 1056 | 894,462 | 2768 s / 32 GB（`nvdla3.ys`） |
| Vortex 1 cluster × 4 核 + L2（vortexgpgpu `50cb1745`） | 13,773,305 | 115 | 1,312,151 | 6048 s / 44 GB |
| BlackParrot 16 核（black-parrot `22548483`） | 5,786,014 | 167 | 242,258 | sv2v 約 83 s，yosys 時間未記 |

成品與 `SOURCES.txt`、`recipes/`（含 `unflop2.py`、`mktop.py`、各設計 `.ys`）在 cthulhu 的 `bench-big/rtl/`；CULS-profiling 的 `outputs/rtl-aig-benchmarks.md` 是中文報告。

## 流程

1. 前處理：SystemVerilog 用 sv2v（BlackParrot、Vortex）；NVDLA 的 `partition_o.v` 是 cpp 風格 `#ifdef`，用 `cpp -P -traditional-cpp -DNVDLA_*_ENABLE`。
2. yosys：`read_verilog` → `hierarchy -check -top` → `synth -noabc -flatten`（NVDLA 用 `synth -run :fine` + `techmap`）→ `dffunmap` → `setundef -zero` → `expose -evert`（見下，要限定範圍）→ `opt_clean -purge` → `write_blif`。
3. `unflop2.py`：yosys 的 BLIF 有 `$_DFFE_`、`$_SDFF_`、`$_DLATCH_`、`$mem_v2`、`$print` 等 subckt，ABC 讀不進去。轉成 `.latch` + 明確的 enable／同步 reset mux；非同步 reset 功能上忽略；`$print` 丟掉；`$mem_v2` 讀資料變 PI；clock／async reset／latch enable 變 PO 再裁掉。
4. ABC：`read_blif; comb; strash; print_stats; write_aiger`。確認 `lat = 0`。

## 坑

- **`expose -evert` 不給選擇會把每條有名字的 wire 變成 PO**：NVDLA 11.4M PO、BP16 約 9.8M PO，AND 數不變但 PO 完全不真實。解法：用 `dump <top>/w:*` 取頂層 output（`mktop.py` 轉成位元清單 `topouts.txt`），`unflop2.py` 只留頂層輸出、flop D、blackbox 接腳；或像 Vortex 只對 RAM 實例 `expose -evert @BB`。
- **`comb` 把 latch D 的 PO 算兩次**：`unflop2.py` 已把 latch D 列為 PO，ABC `comb` 又加一次，兩個 PO 同一個 driver。表面 PO 數 ≈ 2 × latch + 其他。Vortex：`read_blif` 後 `print_stats` 為 `i/o =55931/1313425 lat =1312151`，`comb` 之後 PO 變 2,625,576。拿 PO 數當工作量時要減掉 latch 數；AND／level 不受影響。
- **yosys 0.69 不收「迴圈上界依賴資料」**：NVDLA 的 `vlibs/NV_DW_lsd.v` 要改成常數上界加 `done` 旗標（功能相同）。
- **RAM 要 blackbox**：讀資料→PI，其他接腳→PO。NVDLA 56 種 `nv_ram_*`（`read_verilog -lib`）、BP 的 `bsg_mem_*`、Vortex 的 `VX_sp_ram_asic`／`VX_dp_ram_asic`（原始碼已帶 `(* blackbox *)`，自己加的 `select/blackbox` 沒匹配到只是警告）。沒 blackbox 的小 RAM 會被 yosys 展成 flop + mux tree，是 Vortex 每核約 283K flop 的來源。
- **sv2v 失敗**：BlackParrot 的 `bp_common_rv64_csr_defines.svh` 有具名 `begin/end` 區塊，sv2v 0.0.13 不行，拿掉。`bsg_mesh_stitch` 越界 range select 讓邊界 NoC 位元變 undef，`setundef -zero` 補 0。
- **Vortex 的 `hw/VX_config.vh` 是產生檔**，要先跑 `ci/gen_config.py`（等同 `./configure`），否則 verilator 預處理找不到 include；失敗後要刪掉殘留的 `*_Vortex` 目錄，不然 make 以為已完成。
- **Vortex 的 `hierarchy` 階段約 15 分鐘看起來卡住**（AST derive，generate 很多），其實只是慢，別 kill。整個 yosys 單執行緒。
- **ABC 警告「Constant-0 drivers added to N non-driven nets」**：NVDLA 29,103、BP16 92,576、Vortex 15,322，是原 RTL 沒接的 net 補 0。
- **共用主機的程序管理**：不要 `pkill -x yosys`（會殺到別人的）；等待用 PID，不要 `pgrep -f`（會比對到自己的 shell）；長工作用 `setsid nohup`。ssh 長等待可能 exit 255 中斷，重連後用實際檔案狀態確認，不要信舊的背景 task 狀態。
- cthulhu 與 mazu 不能互相 scp，要經 Mac 中轉。

## 限制

這是 flop／RAM 邊界之間的組合邏輯，不是功能等價網表：async reset 忽略、RAM 黑盒、undriven 補 0。沒做 RTL 對 AIG 的 CEC（BLIF 4–5 GB 太大）。適合做 QoR／速度 benchmark，不適合功能驗證。

## 放大原則

只用設計自己的參數放大（核心數、thread／warp 數等），不用 ABC `double`；Vortex 約 3.1M AND／核。BlackParrot 4 核遠低於 5M AND，要 16 核才夠。
