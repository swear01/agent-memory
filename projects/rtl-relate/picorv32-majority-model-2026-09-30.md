---
title: PicoRV32 多數 assertion 行為候選與無 Formal 階段
scope: project
project: rtl-relate
status: active
updated: 2026-09-30
---

# PicoRV32 majority-behavior candidate

使用 pinned PicoRV32 `ef203c2b0a3fb793280f5114941416c425c5b461`、riscv-formal `c992aa61fdfe0846c5ed90324c596202a1c69b76`。`checks.cfg` 生成 86 個 assertion task（37 RV32I、8 M、25 C、6 register/PC/order、10 CSR）與 1 個 cover；另外 3 個手寫 `.sby` 不在本輪分母。80% 的研究目標若按 86 個 task 計數，需要至少 69 個，但 task 高度相關，名稱比例不等於行為覆蓋率。

新案例在 `experiments/picorv32_majority/`。候選 01 由當前 Codex 助手直接編輯，保留原核心非 M 路徑，讓 `WITH_PCPI=0` 並移除乘除法 PCPI 單元與本 wrapper 不用的 adapter module；檔案從 3,049 縮為 2,103 行。37 RV32I、25 C、10 CSR 共 72 個 task 的相關程式碼路徑仍可見，8 個 M task 明確失去，6 個跨指令檢查仍待判讀。**沒有證明 72 個 task 都保留，也沒有得到實測 80%**；含 M 前史的合法行為不在候選中。大部分減少的行數是原 wrapper 不用的 module，不能當成 solver 問題變小的證據。這輪依使用者要求沒有執行 Formal、編譯或模擬。

兩次獨立 API 生成都未交付候選：DeepSeek Flash gateway 3 次嘗試後 HTTP 403；Muse Spark 1.3 Contributor 的 16,384 completion tokens 幾乎全用於 reasoning，回覆 `finish_reason=length` 且 `content=null`。因此候選 01 不是乾淨 session 的自由生成樣本，不應當作外部模型能力的成功率資料。公開研究素材在 feature branch `feat/picorv32-majority-20260930`，commit `bed50431f7ebff0c35bde8e9a30db1c6eec5f766`。因專案 PR CI 會自動執行 Formal，本輪只推 branch，未開 PR。
