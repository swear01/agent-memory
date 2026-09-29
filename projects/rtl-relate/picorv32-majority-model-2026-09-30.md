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

## Design Compiler mapped-cell follow-up

後續分支 `feat/picorv32-mapped-20260930`（commit `50b74b59ff664935b1d6f98f7b8bf771bfc49cc2`）cherry-pick 前述研究素材，再加入可綜合候選 03–05、DC Tcl、VCS 小型模擬與 `experiments/picorv32_majority/mapped_size.md`。03–05 是 Codex 根據上游已有設計選項直接構造的工程對照，不是獨立 freestyle 模型產出。分支已推送；專案 PR CI 會自動跑 Formal，因此在使用者「先不要任何 Formal」的要求下仍未開 PR。

mazu 的 Design Compiler W-2024.09-SP4 由 `source /apps/eda/synopsys/synthesis.sh 2024.09-sp4` 啟用。用 `/apps/cad/cell_library/CBDK45_FreePDK_TSRI_v1.1/lib/freepdk45_v1.1_t25.db`、20 ns clock、相同 PicoRV32 參數、`compile -map_effort medium -area_effort high -ungroup_all`，`report_area` 與 `get_cells -hierarchical` 統計一致，且各量測均無 macro/black box。

- 不含 RVFI 的實際算術設定：原版 30,108 mapped cells，候選 05（移除 M＋單埠暫存器檔）15,636，少 48.1%；library cell area 少 43.5%。
- 有 RVFI、真實算術設定：32,595 → 17,982，少 44.8%。
- 上游 RVFI/ALTOPS 設定：21,668 → 17,982，少 17.0%。候選 04 保留 M 並改 8 步迭代乘法，在此設定反增至 21,829 cells；不可用實際算術版的減幅代替上游驗證模型的減幅。
- 候選 02 的 `$anyseq` 被 DC 前端以 `VER-110: Function '$anyseq' not defined` 拒絕；它的 Yosys generic-cell 減幅沒有對應的 DC mapped-cell 數。
- VCS 2025.06 指定模擬：候選 05 與原版 7 條非 M 指令的 RVFI 退休紀錄相同；候選 04 的 fast/8 步乘法器在正常與 ALTOPS 模式各 16 組對照一致。沒有跑 Formal、BMC 或 assertion suite，不能宣稱 80% 行為保留。

這是個有用的量測分界：標準 cell 數可在可綜合候選上大幅降低，但若縮減依賴刪掉 M 指令，不能把 cell 改善寫成廣泛 verification 能力；若用 `ALTOPS`，原版乘除法本來就已被簡化，實際算術的面積收益不會等比例出現在驗證模型。
