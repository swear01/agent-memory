---
title: HW-Benchmark 七個 40nm designs 的 ASAP7 與 RAM ROM 整合研究
scope: projects/spec2rtl
status: subset-functional-verified-timing-and-sram-pending
updated: 2026-10-09
---

# ASAP7 benchmark 可行性

研究追蹤：DVLab-NTU/spec2rtl issue #22；SRAM compiler 詳細研究接續 #18 與 `sram-compiler-feasibility.md`。使用者明確把 performance baseline 留待後續；未遷移 production CI、變更 golden、共享 PDK 或 NAS。

## 來源與七個設計

HW-Benchmark 固定 `ed2c429f5cd8a36876081fb0b0b9769935994c3f`，狀態表顯示七個 TSMC 40nm RTL→SYN→SDF gate 完成：Keccak、LBP、BCH decoder、Ed25519、Mamba in-proj、Conv、RLE。這是 upstream snapshot 的完成表，本輪不是重跑所有七個 40nm 流程。

Spec2RTL main `7fc60137674ed02f3b11c2300dce38638b51d7d6`；develop 在研究途中更新為 `25bc7f4b614ab7a11a13845bd337c118c743a82a`（passive CI monitoring）。該版 `scripts/ci/common.py` 仍固定 benchmark `ad97739f1bad6ad78fb7ec5fe12da4ce437718c2`、SDK 1.0.36、Conv/BCH；不能把七個新版 40nm designs 當成既有 CI coverage。

## RAM 與 ROM

- Conv：8×512×8 single-port macro；RLE：12×512×45 single-port macro。最新使用 active-low CEN/WEN、whole-word write，test/EMA/RET/STOV pins；behavioral model initial zero、posedge read/write、write holds Q。原 fixed CI 的 SRAM 契約另論，不能只改 library path。
- 最新 RLE 沒有舊版 per-bit BWEB mask。`top.v` 寫入 `{9'b0, sram_datain}`，只讀低 36 bits；512×36 wrapper 或 512×46 padding 可研究以避開 OpenFinRAM 偶數 width 限制。尚未生成或驗證這個 wrapper，不能宣稱已相容。
- 其餘五個沒有 hard SRAM macro，但有 runtime RTL arrays，仍可能合成成標準元件暫存器。
- 七個均未實例化 hard ROM macro。Keccak `permute.sv` round constants/rho offsets，Ed25519 `constant.vh` 32×16 LUT 是小常數查表，可用標準元件邏輯；並非沒有唯讀資料需求。Mamba in-proj 權重由介面送入，TB readmemh 不等於 DUT ROM。
- 完整 Mamba 不在七個內：`exp_pwl_q4_12_pipelined.sv` 另有兩個 64×16 PWL ROM arrays，還有 unsupported SRAM size blocker。QConv gate timeout、ResAcc reference model、AXI 多 tops 尚未納入。

## ASAP7 真實試跑

Mazu 隔離研究，DC Y-2026.03／VCS 2026.03。OpenFinRAM repaired source `d7e66ff0b20e48498a9c8295ba39a0d8c533185f` 內五個 RVT TT standard-cell DB/LIB，0.7 V／25 C；DC report_units 實測 ns／pF。沿用原 physical clocks與 IO constraints，未找最大頻率。Gate 模型來自固定 ORFS image `sha256:b879915e0ec547a7111e4e765f95b8896b3876d2ee36cf48d139355252678711` 的 ASAP7 stdcell models。202 個 Liberty cell types 均有模型；netlist 無 unresolved references，Keccak 61 種／LBP 33 種 cells。

| Design | RTL checks | ASAP7 synthesis | SDF gate 功能 checks | RTL/gate 相同時間 ns |
| --- | --- | --- | --- | --- |
| Keccak | m00/m01/m11 3/3 | 完成；最小 max-path slack +0.16 ns | 3/3 | 6181／11959／1377 |
| LBP | pat0 1/1 | 完成；最小 max-path slack +5.69 ns | 1/1 | 807071 |

18 個 RTL/TB/pattern 檔案 SHA-256 與 source snapshot 相同。第一輪模型 -v search 沒載入 UDP，失敗；從原始檔取出 14 個相同 UDP definitions 明確 compile 後重跑成功，沒有 stub 或 function 修改。原模型 1ns/10ps 有 delay roundoff，最後用 VCS `-override_timescale=1ns/1fs` 重跑，roundoff warning 消失；保留 `+maxdelays -negdelay +neg_tchk`，沒有 `+notimingcheck`。

**SDF 功能 checks passed，不是完整 timing annotation 或 signoff passed。**

- HAxp5 的 Liberty unconditional A/B→SN IOPATH 對不上模型 conditional specify：Keccak 394／LBP 8 條 `SDFCOM_INF`，對應 arcs 未 annotation。Boolean function 相同，不應因功能通過忽略 delay coverage。
- DFFASRHQN 的 `$recovery/$removal` 無法接受 negative limits：Keccak 6732／LBP 648 條 `SDFCOM_NL`；VCS 建議 `$recrem`。必須校準 model/Liberty timing contract後才可宣稱完整 reset timing checks。
- Keccak 1591 條 `SDFCOM_SWC`：階層 path 非 simple wire，VCS 明確表示 delay still annotated；最終未觀察 runtime timing violation 輸出。
- Ubuntu native工具有 unsupported-kernel、bootstrap缺 csh 警告，但實際 DC 合成與 VCS 執行完成。初始 report_units 放在 current_design 前報 UID-4，後以最終 DDC 重讀、link、report_units 確認單位；保留原始 logs。後續 scripts 已調整 report 位置。

後續應先固定同 release 的 DB/LIB/Verilog／timing model 並嚴格確認 annotation coverage，再接 Conv/RLE wrapper、完整 macro connectivity/DRC/LVS/PEX/characterization，最後補其餘五個 ASAP7 designs。沒有新增 performance baseline、PR threshold 或正式 CI。

ASAP7 是 predictive PDK，優勢是先進節點研究的公開可重現性，不代表比 foundry TSMC 40nm characterization 更準。40nm SS 0.81 V／125 C 與本輪 ASAP7 TT 0.7 V／25 C 不可直接比較 PR PPA。ASAP7 官方 README 與 ORFS stdcell README 為來源。

## 完整 Mamba 的 exp ROM 後續實測

2026-10-09 以相同 HW-Benchmark snapshot 測 `exp_pwl_q12_20_to_q4_12_pipelined`。discretization 算 `A_bar=exp(delta*A)`，此實作用 base/delta PWL 表與線性內插；單 lane 兩張 64×16，共 2,048 bits，預設 tile producer 64 lanes，不能當成整顆 accelerator 總面積。固定 lookup 可合成成標準元件邏輯，不要求 OpenRAM ROM compiler；Mamba 數學需要 exponential，不要求所有實作都用 ROM。

發現原 `initial/$readmemh` 被 DC Y-2026.03 `VER-281` 忽略。原 RTL 69,640 checks passed，原 netlist 在 vector 4096/input `ff7aea43` expected `0001` 得 `0000`，with/without SDF 均重現。這輪 `vcs -R` 的 `$fatal` 仍 return 0，驗證必須要求 PASS marker 並拒絕 Fatal/MISMATCH，不能只信 RC。

改為 128 個原係數的 `localparam` 常數；數值與 `.mem` reference copies 完全相同，arithmetic、pipeline、ports、goldens 未改。HW-Benchmark PR #7 已 merged，patch `e0f608a9cb18d50bf5c985559c7efd473c7b28eb`，main merge `00d21352739089b72d192508148f8fcdb2fc68ff`；讀回 main RTL 與實測 source 相同。最新 head 的 Cursor Bugbot terminal success/no issues found；repo 未配置 GitHub CI workflow，本地 EDA checks 為驗證證據。

- 原與修後 RTL 各 69,640 checks passed：64 segments×64 offsets×16 integer branches，加 8 boundaries 與 4,096 deterministic full-width random。全寬 random 是 preservation，不是對整個 Q12.20 domain 的 exp mathematical accuracy 保證。
- base coefficients 等於 rounded `4096*2^(i/64)`，delta 等於相鄰 quantized base 差分。在被掃描 `[-9,0]` domain 對 rounded float exp 最大 absolute error 1 LSB，未評估整體模型 accuracy。
- 修後 ASAP7：865 standard cells、74 sequential、0 macros/blackboxes；2 ns synthesis constraint report minimum max-path slack +0.04 ns。
- SDF gate：69,640 passed、0 observed runtime timing violations、36 model types resolved；gate bench 4 ns，因 stimuli 在 falling edge 切換，不能當成 2 ns gate qualification。
- 完整六層 Mamba 原始 RTL `pat0` passed、757407 ns、final output bit-exact；未用 report-only demo mode。首輪 Mac tar AppleDouble metadata 阻擋編譯，移除確認為 metadata 的單檔後重跑，未改 design/TB/goldens。

FAx1 有 3,906 missing IOPATH warnings、DFFASRHQN 441 negative recovery/removal warnings，仍非完整 timing annotation/signoff。整顆 Mamba ASAP7 ASIC synthesis/gate simulation、SRAM integration、P&R/signoff/baseline 未完成。研究回報於 issue #22 comment `6067602499`；證據 `<research-root>/outputs/mamba-rom-research-20261009/` 與 Mazu `/var/tmp/mamba-rom-research-20261009`，未部署 NAS。

本機研究證據 `<research-root>/outputs/asap7-benchmark-research-20261009/` 含 source SHA、18-file hash check、commands/results、2.4 MB evidence.tar.gz 及 issue body/readback；Mazu `/var/tmp/asap7-benchmark-research-20261009` 保留完整 models、reports/netlists、失敗與重跑 logs。這是研究 scratch，沒有部署 NAS。
