---
title: RTL relation checker v0 的已驗證界線
scope: project
project: rtl-relate
status: active
updated: 2026-09-20
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

目前 accepted gold 的 J 都是 true。下一個最有判別力的 fixture 是 P2 one-hot → binary，搭正確且非平凡的 inductive invariant；不先擴 LLM 預算。
