---
title: axilxbar freestyle pilot 的 Yosys 前端與 property 接線
scope: project
project: rtl-relate
status: active
updated: 2026-09-28
---

# axilxbar frontend 實測界線

AIsimpV 的 9/30 freestyle pilot 使用 ZipCPU/wb2axip commit `2e8d3bc2d26ddc33d1881022a2a2b9d3f0c16b9b` 的 `axilxbar.v`、`addrdecode.v`、`skidbuffer.v`。系統 Yosys 0.69+post（git `143eb14f9cc55d6f8927e68523b0c9d2166ed02c`）與專案 YoWASP Yosys 0.69 都在這份來源的 `hierarchy -chparam` 路徑觸發 `rtlil_bufnorm.cc:681` 的 `modules_.count(module->name) == 0` assertion。只改 `C_AXI_ADDR_WIDTH=16` 或只改 `OPT_LOWPOWER=0` 都重現。預設參數 4×8、AW32、DW32、LOWPOWER1 不用 `-chparam` 時，`read_verilog -sv -defer ...; hierarchy -check -top axilxbar; proc; check` 成功且回報 0 problems。因此第一輪改用預設組態，未把工具崩潰算成設計不合法。

單獨執行 `read_verilog -formal` 會定義 `FORMAL`，把上游整套 formal code 一起帶入；原題只要一個衍生 property，不能這樣讀 design。已驗證做法：先用 `read_verilog -sv -defer` 讀 design 與依賴，再用另一個 `read_verilog -formal -sv -defer` 讀獨立 property module；在檢查用的 design 副本中實例化該 property，接著 `hierarchy -check -top axilxbar; proc; check; select -assert-any axilxbar_read_hold_property/t:$check`。原版與第一份候選都通過。這只確認接線與 assertion cell 存在，沒有跑 solver。

專案 `scripts/check_freestyle_case.py` 與 `docs/reports/freestyle_rewrite_pilot.md` 保留可重跑命令及第一份候選的證據。第一份模型輸出把完整 crossbar 改成 master0/slave0 read-only stub；獨立分析指出局部化，但人工另發現分析的行號及 upstream FORMAL suite 範圍錯誤。前端 PASS 與一致的自然語言解讀均不能代替原題與候選間的關係檢查。
