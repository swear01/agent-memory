---
title: RTL relation checker v0 的已驗證界線
scope: project
project: rtl-relate
status: active
updated: 2026-09-21
---

# RTL relation checker v0

獨立專案位於 `<project-root>/rtl-relate`，不 import NeuroAbs。目前人工 P1 counter 與 P5 stalled producer 可跑真實 Yosys → BTOR2 → typed IR → Z3 certificate gate、free-nondeterminism property checking、exact concrete replay。這是 tiny feasibility 結果，尚不是 LLM 或加速實驗。

## 需要保留的 soundness 防線

- Frontend 不可丟掉 clock identity。實際反例：C 用 clk，A 改 other_clk，在模型未保存 clock 時仍通過 frozen clk contract。修正是 model 保留經 frontend 驗證的 name/edge，checker 與 contract 精確比對；真 RTL 回歸確認正常 case ACCEPTED、錯 clock ERROR。
- 五項義務必須共用同一 w。P1 observer 不讀 z，無法單獨測到此漏洞；另用 Mealy fixture 令 STEP_MAP 要 z=0、OBS_MAP 要 z=1，確定不存在共同 witness。
- J=(x<2) 搭錯 w=0 可讓 P1 STEP_MAP 成立，但 STEP_J 必須拒絕。空 initial set 也會令所有 implication 空泛成立，必須先獨立檢查非空。
- Property 不收 h/J/w；每拍 z 使用 fresh symbols。SAFE 只來自完整一步證明或完整 reachable closure；有限探索達上限回 UNKNOWN。
- Exact replay SAT 只表示 prefix FEASIBLE，還須確認違反 frozen property 才是 BUG。P5 bug 的第一條 abstract CEX 可能 spurious；若另由 concrete search 找到不同 trace，必須明示其來源，不能冒稱原 trace 已具體化。
- Concrete fallback 的 SAFE/UNKNOWN/ERROR 也要算成本；不能只在找到 bug 時計費。Gate rejection 與 exception 也保存耗時。

## 可重現工具路徑

Checker 僅 Python standard library + Z3 CLI。YoWASP Yosys 可隔離安裝在 `<task-worktree>/.tools/yosys-venv`，不需要 sudo。已測 yowasp-yosys 0.69.0.0.post1233、Yosys 0.69 / 9f75ca1f9、Z3 4.15.4。

重跑介面：`python3 -m unittest discover -s tests -v`；`python3 -m rtl_relate demo --out results/<new-run>`。每次用新目錄，保存 hashes、SMT queries、raw logs、RTL exports、traces 與成本。

目前 accepted gold 的 J 都是 true，one-hot → binary 搭非平凡 inductive invariant 仍是必要控制案例。2026-09-21 使用者已擴大範圍：公開 skidbuffer 8/32-bit、pipeline32 及人工 FSM；固定 C/A 找 certificate 與 joint RTL＋certificate 兩個 LLM pilots 都是必要交付，各最多 4 次／15 分鐘。這些新工作仍未執行，不可混入既有 tiny 成功結果。

## 外部 benchmark 接入陷阱

- `aman-goel/avr` 固定 `9a76dc632066c4416cebccda3a4974a4f8adede8`：`tests/opensource/h_Arbiter/main.v` 的 client 把 `rand_choice=0`，且 `req=0`、`state=NO_REQ` 初始化，請求永不發生；不可當作非平凡正例。改為自由輸入會改變 benchmark，須另標變體。
- 同版 AVR pipeline 的實際檔案是 `tests/opensource/pipeline/pipeline.v`：96-bit datapath 加 64-bit monitor。保留原生 assertion 的現有 export 因 `$check` 失敗，尚未接通；不能刪除 assertion 後冒稱已驗證原 property。
- `ZipCPU/wb2axip` 固定 `2e8d3bc2d26ddc33d1881022a2a2b9d3f0c16b9b`：原始 `rtl/skidbuffer.v`、預設 `DW=8, OPT_OUTREG=1, OPT_INITIAL=1`、未定義 `FORMAL` 時，現有 frontend 匯出 18 state bits，`i_reset` 為普通 public input。這只證明 frontend 接入；原 `FORMAL` assumptions、reset contract、certificate 與 property 都未因此獲證。原始 `rtl/sfifo.v` 則為 `UNSUPPORTED: $memwr_v2`；預設非同步讀取配置的 memory 未初始化，不可補零來讓接入通過。

## 公開專案與協作入口

公開 repo 為 `swear01/rtl-relate`；首次 push 前已完成 Gemini Code Assist linking。`docs/wednesday_plan.md` 是新排程，issues #1–#12 保存依賴／檔案責任／驗收證據；#1 總追蹤、#2 固定來源與 contracts、#12 為不計入必要里程碑的 FIFO。

GitHub Actions `Verify` 已在 Ubuntu 24.04 / Python 3.12 實跑 89 項測試與完整 8 列 demo，保存 `verification-evidence` artifact；不是只有本機 3.14 結果。用 `unittest ... | tee ...` 保存輸出時，job 明設 `defaults.run.shell: bash` 以啟用 pipefail，避免 tee 遮住測試失敗。
