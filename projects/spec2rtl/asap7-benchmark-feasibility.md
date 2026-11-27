---
title: HW-Benchmark 七個 40nm designs 的 ASAP7 與 RAM ROM 整合研究
scope: projects/spec2rtl
status: subset-timing-verified-full-mamba-and-sram-pending
updated: 2026-10-09
---

- **七個 40nm 設計被當成既有 ASAP7／CI 涵蓋。** upstream snapshot 的七個完成項是 Keccak、LBP、BCH、Ed25519、Mamba in-proj、Conv、RLE；現有 CI 固定較舊 benchmark 與 Conv/BCH。隔離 ASAP7 實測是 Keccak 三組、LBP 一組、Mamba Exp 單 lane，非完整 Mamba。ASAP7 predictive PDK 的 RVT TT 0.7 V／25 C 與 foundry 40nm SS 0.81 V／125 C 不能視為等條件 PPA。
- **Gate 功能通過卻漏 timing arcs 或標錯頻率。** 鎖同一套 DB/LIB、Verilog/UDP 和 SDF；定 `current_design` 後用 `report_units` 查 ns／pF。VCS 明確編譯原始 UDP，使用 `-override_timescale=1ns/1fs +maxdelays -negdelay +neg_tchk`，不關 timing check。固定版 FAx1／HAxp5 的 IOPATH 不匹配以排除 cell 處理（DC `set_dont_use [get_lib_cells */FAx1_ASAP7_75t_R]` 不帶 true），會改 cell-policy 基準。DFFASRHQN 改 `$recrem` 並保留負限值；NTCDNC 只有相同 Liberty table 證據時沿已驗證 converter 補回冗餘 constraints。保留 reset-held safe、release 恰違規、data setup 真違規等正負控制。校準後 Exp 69,640 gate checks 在 **4 ns**，2 ns 是 synthesis constraint，不是該頻率 gate qualification。
- **Mamba Exp RTL 對、netlist 卻漏 ROM 值。** DC Y-2026.03 `VER-281` 忽略 `initial/$readmemh`，無 SDF 也會 mismatch。HW-Benchmark PR #7 已改等值 128 個常數（兩張 64×16 table），merged main `00d21352739089b72d192508148f8fcdb2fc68ff`；舊 branch 才需要搬 patch，勿重複編同一 module。驗原係數及 bit-exact PASS／無 Fatal/MISMATCH，VCS `$fatal` 曾 exit 0。每 lane 的 table 不能算全晶片面積；有限掃描的 1 LSB 不外推整體模型。其餘 in-proj TB 的 readmemh 不構成 DUT ROM；小固定 LUT 可由標準元件實作。
- **全 Mamba 合成省略 simulation DRAM 後留下 unresolved module。** `aoc_dram_model.sv` string 使 DC VER-700；直接省略檔案會少硬體 DRAM ports/boundary。之後 local-buffer elaboration 的 exit 137 未證 kernel OOM。需要真正拆硬體界面、做 gate harness、提供合格同製程多 port／masked SRAM。Conv／RLE 的新模型為 512×8／512×45、active-low whole-word，最新版 RLE 無 BWEB，低 36 bit wrapper 仍待驗。詳見 [SRAM qualification](sram-compiler-feasibility.md)。
- **共用套件可讀或對照 bench 綠了，被升成預設。** `/apps/cad/cell_library/ASAP7_EDK/memory-20261009/README.md` 為研究整合入口，含 ROM、校準模型、驗證程序及 RAM 的明確資格；`orfs-platform/` 是固定 ORFS v1.7 副本。從共用唯讀處複製到 scratch 重跑 ROM RTL／SDF、converter 比對和 RAM 反例，依登入帳號檢查讀取及 `verification/SHA256SUMS`，不能以 mazu 可讀替代其他主機。全晶片 P&R／corner coverage、完整 Mamba、SRAM qualification 和 performance baseline 均未就緒；維持既有 production CI/goldens。

[完整原文與實測紀錄](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/projects/spec2rtl/asap7-benchmark-feasibility.md)
