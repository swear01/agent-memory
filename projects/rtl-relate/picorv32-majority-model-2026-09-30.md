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

兩次獨立 API 生成都未交付候選：DeepSeek Flash gateway 3 次嘗試後 HTTP 403；Muse Spark 1.3 Contributor 的 16,384 completion tokens 幾乎全用於 reasoning，回覆 `finish_reason=length` 且 `content=null`。不能把兩份助手整理的候選當作外部模型能力的成功率資料。第一版研究素材曾推到 `feat/picorv32-majority-20260930`（commit `d213ee6aa0c3d4635482ffbd0aefd4cd78127300`）。當時因 PR CI 含 Formal 而未開 PR；後來確認那是專案既有案例的回歸，並非對此候選執行 Formal，把「候選先不跑 Formal」當作不能開 PR 是過度解讀。

## Design Compiler mapped-cell follow-up

後續分支 `feat/picorv32-mapped-20260930`（commit `50b74b59ff664935b1d6f98f7b8bf771bfc49cc2`）cherry-pick 前述研究素材，再加入可綜合候選 03–05、DC Tcl、VCS 小型模擬與 `experiments/picorv32_majority/mapped_size.md`。03–05 是 Codex 根據上游已有設計選項直接構造的工程對照，不是獨立 freestyle 模型產出。

CI 範圍已由 PR #28 調整並合併：`.github/workflows/verify.yml` 移除 feasibility demo、公開 RTL rewrite matrix 與 FIFO assessment，只保留單元／前端測試與測試結果 artifact。`tests/test_formal.py` 等既有 Formal 單元測試**仍在 CI**，因此「CI 不再跑 rewrite 實驗」不等於「CI 完全不跑 Formal」。此 PR/CI 沒有對新 PicoRV32 候選跑 Formal；後續的單獨探測見下節。

研究素材其後由 PR #29 合併到 `main`（merge commit `5f502f28ed9776b069cb9687bdf38d75956d3ebe`），靜態對照頁位於 `experiments/picorv32_majority/index.html`。最新 head 的 GitHub CI 與 Swear Review 均通過；Gemini 重審沒有新意見。本次合併流程只重跑了 VCS 七條非 M 指令 smoke test 與 DC 映射，未在 PR 內做候選 Formal 檢查。
PR #30 修正 `mapped_size.md`：原始 DC/VCS log 是重跑命令的本地輸出，沒有隨 repo 發布。

mazu 的 Design Compiler W-2024.09-SP4 由 `source /apps/eda/synopsys/synthesis.sh 2024.09-sp4` 啟用。用 `/apps/cad/cell_library/CBDK45_FreePDK_TSRI_v1.1/lib/freepdk45_v1.1_t25.db`、20 ns clock、相同 PicoRV32 參數、`compile -map_effort medium -area_effort high -ungroup_all`，`report_area` 與 `get_cells -hierarchical` 統計一致，且各量測均無 macro/black box。

- 不含 RVFI 的實際算術設定：原版 30,108 mapped cells，候選 05（移除 M＋單埠暫存器檔）15,636，少 48.1%；library cell area 少 43.5%。
- 有 RVFI、真實算術設定：32,595 → 17,982，少 44.8%。
- 上游 RVFI/ALTOPS 設定：21,668 → 17,982，少 17.0%。候選 04 保留 M 並改 8 步迭代乘法，在此設定反增至 21,829 cells；不可用實際算術版的減幅代替上游驗證模型的減幅。
- 候選 02 的 `$anyseq` 被 DC 前端以 `VER-110: Function '$anyseq' not defined` 拒絕；它的 Yosys generic-cell 減幅沒有對應的 DC mapped-cell 數。
- VCS 2025.06 指定模擬：候選 05 與原版 7 條非 M 指令的 RVFI 退休紀錄相同；候選 04 的 fast/8 步乘法器在正常與 ALTOPS 模式各 16 組對照一致。沒有跑 Formal、BMC 或 assertion suite，不能宣稱 80% 行為保留。

這是個有用的量測分界：標準 cell 數可在可綜合候選上大幅降低，但若縮減依賴刪掉 M 指令，不能把 cell 改善寫成廣泛 verification 能力；若用 `ALTOPS`，原版乘除法本來就已被簡化，實際算術的面積收益不會等比例出現在驗證模型。

## 候選 05 的 Z3 formal 探測

後續在 `feat/picorv32-mapped-20260930` 對 pinned 生成的 `.sby` 保留 checker、wrapper、深度及 defines，只把原版 RTL 路徑換成候選，並因本機無相容 Boolector 將 `smtbmc boolector` 換成 `smtbmc z3`。SBY 0.69、Yosys 0.69+post、Z3 4.15.4；原始 task 的 `expect pass,fail` 使 FAIL 也可能 exit 0，必須讀 `status`。完整 task、log、trace 和 `summary.json` 當時寫入該工作樹被 Git 忽略的 `results/picorv32_majority/formal-20260930/`；合併後清理工作樹時，這些原始檔隨之移除，未納入版本控制。以下只保留結果摘要，需要原始證據時須重跑。

候選 05 的 10 個 CSR BMC：4 個 `csr_ill_*` PASS，`csrc_inc_{mcycle,minstret}`、`csrc_upcnt_{mcycle,minstret}`、`csrw_{mcycle,minstret}` 共 6 個 FAIL；這 6 題原版在相同 Z3 設定下全部 PASS。`csrw_mcycle_ch0` 對照中，單埠候選 03 PASS、只刪 M 的候選 01 FAIL；兩個 mcycle `csrc` 題也同樣是候選 03 PASS、候選 01 FAIL。這定位到刪 M／PCPI，而非單埠暫存器。候選 05 的 ADD、MUL、C.ADD、REG、PC-forward 五個代表 task 在 150 秒內未結束，不能算 PASS/FAIL。尚未量完整 86 題的保留率。

`csrw_mcycle_ch0` 反例於第 15 拍退休 `0xc80030f3`（CSRRC x1, cycleh, x0），RVFI 回報 trap，但 checker 要求合法讀取不 trap；自訂 cover 確認原版、候選 03、候選 05 都可在第 15 拍退休另一條不 trap 的 `c80` CSR 指令，故不是整類 CSR 完全不可達。核心根因：候選把 `WITH_PCPI` 固定為 0，不只刪 M 算術，還繞過上游對不認得指令的 PCPI 等待／timeout 路徑，讓部分 trap 更早進入 RVFI；上游原版 `WITH_PCPI=1` 是由 wrapper 啟用的 M 引擎帶來。這是固定深度 BMC 的實際 regression，不足以單獨判斷所有 CSR 語意或 80% 行為保留。

## 直接使用 JasperGold 的對照

mazu 的 JasperGold 2025.03 可直接跑此 RTL/SVA，不需要 Yosys 或 Boolector。先 `source /apps/eda/cadence/jasper.sh 2025.03`；由於主機 Ubuntu 26.04 不在此版支援清單，啟動時需 `jg -fpv -batch -no_wait -allow_unsupported_OS -proj jgproject -tcl run.tcl`。`run.tcl` 以 `analyze -sv csrw_mcycle_ch0.sv wrapper.sv picorv32.v`、`elaborate -top rvfi_testbench`、`clock clock`、`reset reset`、`prove -all`、`report` 執行；`.sby` 不能直接當作 Jasper Tcl，必須取出相同 defines/checker。單一 `csrw_mcycle_ch0` checker 的原版報告為 15/15 assertions proven、0 cex；候選 05 為 10 proven、5 cex，反例深度均為 15。兩邊也都有第 15 拍可達的 assertion precondition；原版另有 3 個不可達的 precondition cover。證據保存在 `<project-root>/results/picorv32_majority/jasper-direct-20260930/{original,candidate05}/`（被 git 忽略）。原版 elaboration 有乘法運算自動 black-box 警告，wrapper 兩版都有未驅動記憶體輸入警告；此對照驗證 Jasper 可執行且重現此 checker 的差異，但不是完整 86 題保留率。

後續完整 86 題優先用 JasperGold。上游 `rvformal_rand_const_reg` 在 Jasper 非 YOSYS 巨集路徑只是未驅動 `reg`，須對 checker 的 `insn_order`／`register_index` 加 Jasper 原生 `assume -constant checker_inst.<signal>`，才保留「值任意但整條 trace 固定」語意。漏掉時六個 register／PC／order 原版題全部出假反例；補上後原版 **86/86 bounded pass**，候選 05 **72/86 bounded pass、14 題反例**。14 題為刪除 M 導致的 8 題 M，以及額外 6 題 CSR（四個 `csrc_*` 和兩個 `csrw_*`）。候選通過的 62 題非 M 指令之 check-cycle 非 trap cover 全部可達，另 10 題 CSR／跨指令通過題的 assertion precondition 可達。每題按 pinned check cycle 15／20／25／30 做 60 秒 Jasper proof，沒有 timeout 或工具錯誤。`72/86=83.7%` 達 69 題數字門檻，但只是有限深度 assertion-task 通過率，**不是 83.7% 原版行為包含或等價證明**。專案已把 JasperGold 作為 RTL assertion 驗證優先工具；完整方法、逐題 JSON 和重跑腳本見 `<project-root>/experiments/picorv32_majority/`，原始 log 留在 Git 忽略的 `<project-root>/results/picorv32_majority/`。

同一批 Jasper log 的時間對照：只比較原版與候選 05 **都 pass 的 72 題**，以各題 `jg_session_0.log` 的 `# started` 到 `console.txt` 的 `Printed on` 作為 session 啟動到報表時間（秒級，含分析／展開／證明，未含最後退出），中位數 **25 → 20.5 秒**，72 題加總 **1920 → 1471 秒（少 23.4%）**；56 題較快、3 題較慢、13 題秒級持平。從 engine `Exited with Success (@ … s)` 取每題最後完成的 proof 時間，中位數 **22.01 → 17.82 秒**，加總 **1713.61 → 1270.01 秒（少 25.9%）**。RV32I 37 題 session 加總 1216 → 909 秒，C 25 題 603 → 460 秒；4 題通過的 CSR 與 6 題跨指令幾乎無 session 收益。14 題候選 FAIL 的 session 加總 355 → 84 秒，主要因很快找到反例，不能算成功驗證的加速。runner 當時未直接量完整 process 或整批 wall time，且四題並行、原版與候選先後執行，所以這些是單次 log 的每題時間對照，不能推論整批交付時間或穩定多次量測的加速；映射 cell 數減幅也不可拿來代替 formal runtime。
