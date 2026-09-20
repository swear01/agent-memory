---
title: AIsimpV RTL certificate checker 的已驗證界線
scope: project
project: AIsimpV
status: active
updated: 2026-09-21
---

# AIsimpV RTL certificate checker

獨立專案位於 `<project-root>/rtl-relate`，不 import NeuroAbs。目前人工 P1 counter 與 P5 stalled producer 可跑真實 Yosys → BTOR2 → typed IR → Z3 certificate gate、free-nondeterminism property checking、exact concrete replay。既有 tiny 回歸保留；2026-09-21 已另完成四題公開 RTL／FSM 人工改寫矩陣，但未證成驗證加速。

## 需要保留的 soundness 防線

- Frontend 不可丟掉 clock identity。實際反例：C 用 clk，A 改 other_clk，在模型未保存 clock 時仍通過 frozen clk contract。修正是 model 保留經 frontend 驗證的 name/edge，checker 與 contract 精確比對；真 RTL 回歸確認正常 case ACCEPTED、錯 clock ERROR。
- 五項義務必須共用同一 w。P1 observer 不讀 z，無法單獨測到此漏洞；另用 Mealy fixture 令 STEP_MAP 要 z=0、OBS_MAP 要 z=1，確定不存在共同 witness。
- J=(x<2) 搭錯 w=0 可讓 P1 STEP_MAP 成立，但 STEP_J 必須拒絕。空 initial set 也會令所有 implication 空泛成立，必須先獨立檢查非空。
- Property 不收 h/J/w；每拍 z 使用 fresh symbols。tiny backend 的 SAFE 來自完整一步證明或完整 reachable closure；有限探索達上限回 UNKNOWN。公開 RTL backend 的 SAFE 需要 SMTBMC base 與 k-induction 都通過，單獨 BMC 通過只能是 BOUNDED。
- Exact replay SAT 只表示 prefix FEASIBLE，還須確認違反 frozen property 才是 BUG。P5 bug 的第一條 abstract CEX 可能 spurious；若另由 concrete search 找到不同 trace，必須明示其來源，不能冒稱原 trace 已具體化。
- Concrete fallback 的 SAFE/UNKNOWN/ERROR 也要算成本；不能只在找到 bug 時計費。Gate rejection 與 exception 也保存耗時。

## 可重現工具路徑

Checker 僅 Python standard library + Z3 CLI。YoWASP Yosys 可隔離安裝在 `<task-worktree>/.tools/yosys-venv`，不需要 sudo。已測 yowasp-yosys 0.69.0.0.post1233、Yosys 0.69 / 9f75ca1f9、Z3 4.15.4。

重跑介面：`python3 -m unittest discover -s tests -v`；`python3 -m rtl_relate demo --out results/<new-run>`。每次用新目錄，保存 hashes、SMT queries、raw logs、RTL exports、traces 與成本。

2026-09-21 正式人工矩陣 `wednesday-20260921-final01`：FSM、skid8、skid32、pipeline32 共 12 基本 attempts，全數符合 good SAFE／coarse SPURIOUS_TRACE／錯證書拒絕。FSM one-hot→binary 以非平凡 J 限制合法狀態，正反向證書均通過；skid J 為 r_valid ⇒ output_valid。公開部分只有 2 個家族、3 個實例，不能當成 4 個公開家族。重跑介面：`python3 -m rtl_relate.wednesday --out results/<new-run>`。LLM 固定 C/A 與 joint 兩條 pilot 各最多 4 次／900 秒；其結果需查獨立 ledger，不可把人工證書成功歸給 LLM。

## 外部 benchmark 接入陷阱

- `aman-goel/avr` 固定 `9a76dc632066c4416cebccda3a4974a4f8adede8`：`tests/opensource/h_Arbiter/main.v` 的 client 把 `rand_choice=0`，且 `req=0`、`state=NO_REQ` 初始化，請求永不發生；不可當作非平凡正例。改為自由輸入會改變 benchmark，須另標變體。
- 同版 AVR pipeline 的實際檔案是 `tests/opensource/pipeline/pipeline.v`：96-bit datapath 加 64-bit monitor。原生 assertion 的一般 export 會遇到 `$check`。目前只對固定 source hash 接入受信任 extraction：保留原 history monitor，匯出原 prop 檢查等價，再以同一 frozen predicate 建立外部 harness。不可一般化成刪掉任意 assertion。
- `ZipCPU/wb2axip` 固定 `2e8d3bc2d26ddc33d1881022a2a2b9d3f0c16b9b`：原始 `rtl/skidbuffer.v`、預設 `DW=8, OPT_OUTREG=1, OPT_INITIAL=1`、未定義 `FORMAL` 時，現有 frontend 匯出 18 state bits，`i_reset` 為普通 public input。v1 明確把同步 reset 保留為 unconstrained public input；實驗驗證衍生 hold property，不載入上游完整 `FORMAL` assumptions，不能宣稱完整 upstream suite 已通過。原始 `rtl/sfifo.v` 則為 `UNSUPPORTED: $memwr_v2`；預設非同步讀取配置的 memory 未初始化，不可補零來讓接入通過。

## 公開專案與協作入口

公開 repo 已依工作資料夾名改為 `swear01/AIsimpV`；舊名會 redirect，remote／active links 已更新，改名後也成功 `gemini-link swear01/AIsimpV`。本機子目錄與 Python package `rtl_relate` 沒有為此做無關改名。`docs/wednesday_plan.md` 是新排程，issues #1–#12 保存依賴／檔案責任／驗收證據；#1 總追蹤、#2 固定來源與 contracts、#12 為不計入必要里程碑的 FIFO。

GitHub Actions `Verify` 已在 Ubuntu 24.04 / Python 3.12 實跑 89 項測試與完整 8 列 demo，保存 `verification-evidence` artifact；不是只有本機 3.14 結果。用 `unittest ... | tee ...` 保存輸出時，job 明設 `defaults.run.shell: bash` 以啟用 pipefail，避免 tee 遮住測試失敗。

## 公開實驗的量測與工具界線

- `h/J/w` 不進 property harness。skid 18→10／66→34 design bits；pipeline 96→64 design bits，64 原 monitor bits 不變。COI 後的 bit/cell 數還含取樣 harness，不能冒稱是純 DUT 規模。
- 正式 run 的 good workflow 約 1.43–1.92 秒，正常 B0 約 0.54–0.60 秒，單次結果沒有加速；人工準備時間未量測。失敗 attempts、reverse certificate、cover/strictness/replay 都算 machine cost，但工程開發回歸不混入該正式矩陣。
- Yosys Witness `bits` 字串須反向對應 LSB-first signal IDs。由 actual input witness 與 typed model 還原輸出後，仍須重新檢查 A-prefix SAT 與 frozen P 違反，再 exact replay C；缺漏 input bit 的 zero completion 必須顯式記錄且重新驗證。
- SMTBMC `requested_depth` 不代表已完成檢查深度；CEX 保留實際 failure step。frontend／formal 子程序用總 deadline，timeout 要終止整個 process group，不能每個階段重領完整總預算。
