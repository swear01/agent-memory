---
title: AIsimpV 通用 decision model 調查與可替換決策點
scope: project
project: AIsimpV
status: active
updated: 2026-10-22
---

# Jev 型通用 decision model 與 RTL abstraction

2026-10-22 文獻／網路調查，**沒有跑任何模型或 solver 實驗**。使用者想評估把 RTL abstraction pipeline 中的選擇題交給「Jev 型」decision model（輸入 context＋有限選項，輸出選項機率分布，不生成文字）。

## 外部現況（多為 preprint 或廠商數據）

- 沒找到任何針對 Verilog／RTL／EDA 的 decision model，也沒有 RTL 上的評測。JEVal（arXiv `2610.03935v2`，36 datasets、10 domains）**沒有 code／engineering domain**；專業領域（醫、法、金融）decision model 比最佳生成式 LLM 低 3–6 點，可預期 RTL 也有 domain gap。
- JEVal choice＋noul 準確率：SII-Decision-27B 79.24%（ECE 0.0149 全場最低）、Jev 78.38%、Kev-27b 77.17%、Nimble-9b 73.87%、SII-Decision-4B 71.92%；非 decision 的 GPT-6-Luna non-thinking 80.93% 但 ECE 0.1159。
- 延遲：SII-Decision-27B 單張 H100 median 約 110 ms、≤1K tokens 約 0.2 s，但約 2K tokens 後比 Jev 慢；SII-4B 約 60 ms。Jev 是 hosted API，Red Hat 量 348／360 ms median，另有 Cloudflare 數據約 524 ms。
- 限制：JEVal 輸入上限 20,480 tokens、選項上限 255；Jev 長 context 準確率由 4–8K 的 93.5% 降到約 20K 的 74.5%；Jev 機率過度自信。
- LLM-as-Jev（arXiv `2610.02076`）：免訓練地用 next-token 機率讀出編號選項，Qwen3.5-4B 可追平同 backbone 的 Jev 型模型；小模型可用 KL-anchored fine-tune。這是把 code 強的 LLM 變成 RTL 專用 decision model 的可行路線。

## Hosted 廠商比較（使用者放寬到 2 秒、不本地跑）

- 沒有公認單一榜首；各家多用自家 benchmark。較中立的兩個榜結論不同：
  - JevBench（benchmarkheaven.com，board v1.7.46，2026-10-21）官方 Score 混入 speed／cost。只看 Intelligence，decision model 最高為 Sqwish `kettei-fat` 80（Jev 1.13.0 為 64）；唯一的 code 數據為「Coding & software」kettei-fat 86.5、Jev 64.3。kettei 文件未列 context 上限，新帳號可能需 waitlist。
  - Artificial Analysis Decision Index v1.0（2026-10-13，無 code 類任務）：Jev 1.13 為 58，是文中最高的 decision model；Claude Haiku 5.5 (Low) 61、GPT-6 Luna (Low) 55 為語言模型。
- 長 context：Jev 32K、Cloudflare Clef 64K（自報 Decision Index 61.2）、Perplexity pplx-decider v1.1 約 250K（自報 61.56，open weights＋API）。
- 放寬到 2 秒後，低 effort 生成式 LLM 也可入選；JevBench 上 GPT-6 Luna low 的 Intelligence 97、p50 1.85 s，準確度高於所有 decision model，代價是成本。實驗應把它列為對照組。

## 對本專案的含意

- 既有 bundle 很大（PicoRV32 majority prompt 84,726 tokens），整份 RTL 直接餵 decision model 會超過 20K 上限且遠超 500 ms；必須先由 Yosys／COI 抽成每個選項的短特徵（訊號寬度、fanin/fanout、與 assertion 距離、CEX 摘要）。
- 每次 solver 呼叫是秒級（0.3–60 s），單次決策 500 ms 不是瓶頸；延遲只在「每個訊號／每個 proof-tree node 都要決策」的大量呼叫時才重要。
- Decision model 只能替換「選擇」，不能產生 abstract RTL、h/J/w 證書或 invariant；soundness 仍由 formal gate 判定。
- 舊 SFIFO 的單點 havoc 空間 33/33 全為 CEX（見 [checker 記憶](certificate-checker-v0.md)），在那個空間換更好的選擇器也不會成功；測 cutpoint 選擇需改用確實存在可行 cut 的題目。
