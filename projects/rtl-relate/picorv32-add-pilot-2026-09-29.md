---
title: PicoRV32 ADD freestyle pilot 的上下文與 property 陷阱
scope: project
project: rtl-relate
status: active
updated: 2026-10-01
---

# PicoRV32 ADD pilot

完整上游 PicoRV32 `picorv32.v` 是 3,049 行；riscv-formal 的 `rvfi_macros.vh` 單檔約 780 KB。把整個 macro 一併送給 DeepSeek Flash，使第一輪 request 約 979 KB、API 回報 407,950 prompt tokens。macro 對 formal elaboration 必要，對目標 ADD 改寫的生成上下文卻可排除。保留完整 CPU、wrapper、`rvfi_insn_check`、`insn_add` 和 SBY 設定後，第二輪 request 約 146 KB、回報 56,538 prompt tokens。兩次各一個樣本，不能據此推論上下文縮短對成功率的因果效果。

上游 `insn_add_ch0` 用 RVFI 回報的 `rs1_rdata`、`rs2_rdata` 算出 `spec_rd_wdata`。第二輪候選移除暫存器檔，把兩個來源值與結果都固定為零；原 checker 在這份候選上的 Z3 BMC 約 1 秒 PASS，且 cover 確認第 20 拍 `check && spec_valid && !trap` 可達。它仍不是原 CPU 的有效抽象：非零暫存器來源值及其他合法指令軌跡不再保留，另有七個原始 module 宣告缺失。這是 property PASS 且目標事件可達，仍不能推回原設計的具體例子。

原設計的同一 Z3 BMC 在第 20 拍求解約 5 分 37 秒仍無結論而停止；第一輪候選 120 秒未得結論。上游產生的 SBY 原用 Boolector，本地可攜 task 只換成 Z3 並調整路徑。不可拿候選 1 秒與原版未完成時間宣稱 sound speedup。詳細來源、命令、候選與審查頁見 project `docs/reports/picorv32_pilot.md` 與 `experiments/picorv32/`。

在 riscv-formal commit `c992aa61fdfe0846c5ed90324c596202a1c69b76` 的 `cores/picorv32/checks.cfg` 執行 `python3 ../../checks/genchecks.py`，實際產生 87 個獨立 SBY 任務：70 個指令檢查（RV32I 37、M 8、C 25）、16 個其他 BMC 檢查、1 個 cover。`cores/picorv32/` 另外追蹤 `complete.sby`、`honest.sby`、`cover.sby` 三個手寫任務，不在 87 個生成任務內，也不是 `make checks` 的清單。`reg_ch0` 只在先前寫入和後續讀取條件成立時檢查一致性，無法單獨排除全零候選；對同一份候選跑多個獨立上游 task，並加上非零運算元與相依指令的 reachability 檢查，比把不同 task 的 assumptions 直接合併安全。

## Opus 5.5 medium：單一 assertion 的 partial model 與 core-only 封裝

在相同 frozen `insn_add_ch0`、wrapper 與原始碼下，`claude-opus-5-5` medium 使用額外的激進簡化提示，重寫出 370 行主 `picorv32` module；原主 module 為 2,106 行。原始模型輸出仍含七個未使用的輔助 module（全檔 1,319 行）；後續 core-only 衍生檔只去掉這七個 module、保留原有檔頭與主 module（442 行），已逐位元組比對確認主 module 相同。core-only 和完整候選是替代編譯來源，不能同時編進同一設計。衍生檔已由專案 PR #35 合併到 `main`。與 Sonnet 候選相比，模型、effort 和新增提示同時改變，差異不能只歸因於模型能力。

主核心保留取指、32×32 暫存器檔、部分 RV32I ALU 及 RVFI ADD，移除 compressed、load/store、branch/jump、PCPI M 與 CSR 執行路徑並改變退休時序；介面仍保留供 frozen wrapper 使用。原 checker frontend PASS；同一 Z3 BMC depth 21、skip 20 在模型原版約 2.41 秒 PASS，`ADDI x1,5; ADDI x2,7; ADD x3,x1,x2` 的模擬 RVFI 回報 5+7=12；另以 RVFI 寫回值突變使 checker FAIL，JasperGold 找到第 20 拍非零 ADD cover。core-only 衍生檔另外重跑 frontend 與 Z3 BMC PASS。以上驗證的是**此受限模型的 assertion**，沒有證明原 PicoRV32 的行為包含、等價或 refinement；也不能把原版長時間未完成與此模型短時間 PASS 解讀成同一義務的加速。

相同 Yosys 0.69+post 展開流程下，原始版／Opus 候選 generic cells 為 1,359／248、`$dff` 為 171／31；屬於結構計數，不是標準單元面積或求解難度。審查時還需區分 review workflow 成功執行與 inline finding；對 decoder 的修改建議應先對照 ISA，例如 OP-IMM `funct3=111` 是合法 `ANDI`，不能誤判為非法。公開素材見 project `docs/reports/picorv32_pilot.md` 與 `experiments/picorv32/candidate-opus-5-5-medium/`；完整工具 log 仍在 Git 忽略的結果目錄，沒有隨 repo 發布。
