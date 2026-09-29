---
title: PicoRV32 ADD freestyle pilot 的上下文與 property 陷阱
scope: project
project: rtl-relate
status: active
updated: 2026-09-30
---

# PicoRV32 ADD pilot

完整上游 PicoRV32 `picorv32.v` 是 3,049 行；riscv-formal 的 `rvfi_macros.vh` 單檔約 780 KB。把整個 macro 一併送給 DeepSeek Flash，使第一輪 request 約 979 KB、API 回報 407,950 prompt tokens。macro 對 formal elaboration 必要，對目標 ADD 改寫的生成上下文卻可排除。保留完整 CPU、wrapper、`rvfi_insn_check`、`insn_add` 和 SBY 設定後，第二輪 request 約 146 KB、回報 56,538 prompt tokens。兩次各一個樣本，不能據此推論上下文縮短對成功率的因果效果。

上游 `insn_add_ch0` 用 RVFI 回報的 `rs1_rdata`、`rs2_rdata` 算出 `spec_rd_wdata`。第二輪候選移除暫存器檔，把兩個來源值與結果都固定為零；原 checker 在這份候選上的 Z3 BMC 約 1 秒 PASS，且 cover 確認第 20 拍 `check && spec_valid && !trap` 可達。它仍不是原 CPU 的有效抽象：非零暫存器來源值及其他合法指令軌跡不再保留，另有七個原始 module 宣告缺失。這是 property PASS 且目標事件可達，仍不能推回原設計的具體例子。

原設計的同一 Z3 BMC 在第 20 拍求解約 5 分 37 秒仍無結論而停止；第一輪候選 120 秒未得結論。上游產生的 SBY 原用 Boolector，本地可攜 task 只換成 Z3 並調整路徑。不可拿候選 1 秒與原版未完成時間宣稱 sound speedup。詳細來源、命令、候選與審查頁見 project `docs/reports/picorv32_pilot.md` 與 `experiments/picorv32/`。

在 riscv-formal commit `c992aa61fdfe0846c5ed90324c596202a1c69b76` 的 `cores/picorv32/checks.cfg` 執行 `python3 ../../checks/genchecks.py`，實際產生 87 個獨立 SBY 任務：70 個指令檢查（RV32I 37、M 8、C 25）、16 個其他 BMC 檢查、1 個 cover。`cores/picorv32/` 另外追蹤 `complete.sby`、`honest.sby`、`cover.sby` 三個手寫任務，不在 87 個生成任務內，也不是 `make checks` 的清單。`reg_ch0` 只在先前寫入和後續讀取條件成立時檢查一致性，無法單獨排除全零候選；對同一份候選跑多個獨立上游 task，並加上非零運算元與相依指令的 reachability 檢查，比把不同 task 的 assumptions 直接合併安全。
