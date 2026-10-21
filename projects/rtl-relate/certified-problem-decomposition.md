---
title: AIsimpV LLM 驅動的可驗證問題簡化與分解
scope: project
project: AIsimpV
status: active
updated: 2026-10-21
---

# AIsimpV：讓 solver 解簡化後的新問題

2026-10-21 整理本次對話明確指定的研究方向。這是方向與驗收條件，不是新增實驗結果；既有 FIFO、PicoRV32 等實驗仍依各日期原報告，不被本筆記覆寫。

## 使用者指定的範圍

- 原任務為 `b ⊨ a`。LLM 可同時提出新 design/model `b′`、derived property `a′` 與證書，保證 `(b′ ⊨ a′) ⇒ (b ⊨ a)`；也可拆成多個子題，保證 `(∧ᵢ bᵢ ⊨ aᵢ) ⇒ (b ⊨ a)`。
- 原 design、property、環境、clock/reset 與觀察語意固定；新 property 可改，但必須有原目標成立的充分推論。不能弱化原目標或偷加未證環境假設。
- 重點是 solver 實際解新的小模型／子題，以及完整成本或可解率改善。h/J/w 只是一種證書格式；不以 RTL 行數、state bits 或 gate 數代替成效。
- **Proof lifting／把證明搬回原模型並尋找全域 invariant 的路線，使用者指定交由另一個 session 處理，本線暫不推進。** 局部 contract 仍可包含 invariant，但本線產物是簡化模型、子題和可檢查的分解。
- 候選、contract 與證書由 LLM 提出；不得要求使用者先人工製作有效摘要或 gold certificate，才開始測試。

## 可檢查的分解

`CheckDecomposition(root, children, certificate)=accept` 必須保證：所有必要 child goals 成立，便能推出 root goal。每個 child 明列模型、property、環境與依賴。

最小切點規則：原模型為 `C || M`，`S` 為同介面、同時序語意的 summary。證明 `Traces(M) ⊆ Traces(S)`，再證 `C || S ⊨ P`，便可推出 `C || M ⊨ P`。包含關係須涵蓋所有合法介面輸入；若依賴假設 A，A 也須由原環境或已完成的子題保證。

- LLM 選切點、summary／contract、derived properties，依形式反例、失敗義務與成本修正；solver 完成子題；獨立 checker 核對組合規則與完成狀態。
- AND node 的所有必要子題都需完成；OR node 表示不同完整證明策略擇一成功，不能用來漏掉 case splitting 的情況。
- 不能循環借用未證結論。第一個原型可先限制證明依賴為 DAG；這是建議，不是禁止 RTL feedback。Circular assume–guarantee 須另有正確的初始化／時間歸納規則。
- Case splitting 需完整 coverage；unbounded safety 不能靠有限 BMC 無反例宣布完成。抽象子題的反例須回原模型取得合法違例，才可報 BUG。

## 教師題目與文獻定位

先前核對的教師 Project 06 有 RTL cut abstraction、refinement、反例具體化子題及 AND–OR proof tree，因此本方向與其目標接近；不代表老師已核准具體實作方案。

以下文獻於 2026-09-21 討論核對，這次僅整理，不聲稱重現論文實驗或更新所有版本：

- McMillan，*A methodology for hardware verification using compositional model checking*，2000，DOI `10.1016/S0167-6423(99)00030-1`：以 refinement、temporal case splitting、data reduction、symmetry 產生 SMV 可解的子題。
- Gupta、McMillan、Fu，*Automated assumption generation for compositional verification*，2008，DOI `10.1007/s10703-008-0050-0`：自動合成 assume–guarantee 介面 automaton，並非所有自動 contract discovery 都由 LLM 首創。
- SMASH，*Compositional May-Must Program Analysis: Unleashing the Power of Alternation*，POPL 2010：軟體 procedure 邊界的按需 may/must summaries；RTL 還須處理跨拍與 feedback。
- ConVer，*Using Contracts and Loop Invariant Synthesis for Scalable Formal Software Verification*，arXiv `2605.27051v1`：LLM 提出 C function contracts，ESBMC 驗證局部實作及替換後整體；不能當作本專案 RTL 效果證據。
- 暫緩分支來源：Cabodi 等，*Manipulating Proof Certificate Invariants in the Presence of Model Transformations*，IEEE Access 2026，DOI `10.1109/ACCESS.2026.3687715`。利用新模型 proof 與轉換紀錄搬回 invariant，與直接猜 invariant 有別，但和 invariant 路線重疊。

## 實驗建議與證據界線

先試一個切點、兩到三個必要子題，由 LLM 產生 summary／證書，以現有 solver 完成局部認證及簡化後 property proof；不先建完整遞迴 scheduler。此為建議，本次對話尚未執行。

比較同一凍結任務的直接驗證、固定切點／摘要策略、LLM-guided decomposition。成本包括所有搜尋失敗、前端轉換、局部認證、子題求解及組合檢查；平行 wall-clock 與累計 machine time 分開。Property 很快但認證重證原難題，不能算大幅端到端加速。

有用分解須把跨模組相依壓成便宜且足夠的介面關係；若介面幾乎重現完整實作，可能沒有收益。Soundness 可由規則保證，效率仍需同工具／預算的實測。

詳細筆記：`<project-root>/artifacts/rtl-abstraction-research-2026-09-21/CERTIFIED_PROBLEM_REDUCTION.md`。工程結果見 [checker 記憶](certificate-checker-v0.md) 及本目錄各日期實驗記憶。
