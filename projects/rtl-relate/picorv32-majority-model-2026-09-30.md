---
title: PicoRV32 多數 assertion 行為候選與無 Formal 階段
scope: project
project: rtl-relate
status: active
updated: 2026-09-30
---

# PicoRV32 majority-behavior candidate

使用 pinned PicoRV32 `ef203c2b0a3fb793280f5114941416c425c5b461`、riscv-formal `c992aa61fdfe0846c5ed90324c596202a1c69b76`。`checks.cfg` 生成 86 個 assertion task（37 RV32I、8 M、25 C、6 register/PC/order、10 CSR）與 1 個 cover；另外 3 個手寫 `.sby` 不在本輪分母。80% 的研究目標若按 86 個 task 計數，需要至少 69 個，但 task 高度相關，名稱比例不等於行為覆蓋率。

新案例在 `experiments/picorv32_majority/`。候選 01 由當前 Codex 助手直接編輯，保留原核心非 M 路徑，讓 `WITH_PCPI=0` 並移除乘除法 PCPI 單元與本 wrapper 不用的 adapter module；檔案從 3,049 縮為 2,103 行，但核心模組只從 2,106 縮到 2,019 行。37 RV32I、25 C、10 CSR 共 72 個 task 的相關程式碼路徑仍可見，8 個 M task 明確失去，6 個跨指令檢查仍待判讀。**沒有證明 72 個 task 都保留，也沒有得到實測 80%**；含 M 前史的合法行為不在候選中。

候選 02 選用 PicoRV32 原始碼既有的 `RISCV_FORMAL_BLACKBOX_REGS` 分支：x0 仍為 0，兩個非零來源暫存器的讀值用 `$anyseq` 表示，原本的 M/C/CSR/ALU/RVFI 路徑仍在。它不追蹤跨指令的 register read-after-write 一致性，因此 `reg_ch0` 的核心意義失去，其他跨指令任務仍待審查。這是上游現成抽象機制，**不是 LLM 自發明**；第二份也是當前助手直接整理，不是獨立 API 生成。

只跑 size-only Yosys `synth -flatten; stat`，沒有跑 Formal、BMC、模擬或行為檢查。相同 wrapper 參數與上游 `RISCV_FORMAL`、`DEBUGNETS`、`RISCV_FORMAL_ALTOPS` defines 下，通用 cell 數為原版 12,412、候選 01 11,460（−7.7%）、候選 02 7,221（−41.8%）。`RISCV_FORMAL_ALTOPS` 已使 M 算術較簡單；不帶這些 defines 的一般硬體組態中，候選 01 的 18,823→9,864（−47.6%）不能代表上游 assertion 任務。cell 數不是 solver runtime 或行為涵蓋率。詳細命令見 `experiments/picorv32_majority/structural_size.md`。

兩次獨立 API 生成都未交付候選：DeepSeek Flash gateway 3 次嘗試後 HTTP 403；Muse Spark 1.3 Contributor 的 16,384 completion tokens 幾乎全用於 reasoning，回覆 `finish_reason=length` 且 `content=null`。不能把兩份助手整理的候選當作外部模型能力的成功率資料。公開研究素材在 feature branch `feat/picorv32-majority-20260930`，最新 commit `d213ee6aa0c3d4635482ffbd0aefd4cd78127300`。因專案 PR CI 會自動執行 Formal，本輪只推 branch，未開 PR。
