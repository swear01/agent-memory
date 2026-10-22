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
- 同一個 Luna 有官方 decision mode：OpenAI Decisions API（`POST /v1/decisions`，public beta，只支援 `gpt-6-luna`，question 類型 predicate／choice／score，回傳 probabilities，只收 input token $0.10/M）。官方文件未提 `reasoning.effort`。JevBench 上 Decisions 版 Intelligence 57、p50 0.30 s，遠低於 Luna low effort 生成式的 97／1.85 s；高分來自 reasoning，decision mode 沒有保留。同 backbone 的兩種模式是 RTL 決策實驗的乾淨對照。

## 他人的研究方法（可沿用）

- JEVal 把每題轉成 state／instruction／criteria；生成式 LLM（含 thinking）必須輸出嚴格 JSON：答案＋每個候選的機率，無效輸出算錯。SII-Decision 用 Reasoning-to-Readout Self-Distillation，把 thinking LLM 蒸餾成第一個 token 的讀出；student 有時不及 teacher（如 TruthfulQA）。
- LLM-as-Jev：選項編號、prefill `Best answer: [`、以候選 suffix 的 log-prob 做 softmax；以人工標註公開資料做 KL-anchored LoRA。收益隨監督覆蓋而變；只訓練窄任務會產生捷徑偏差；只用短 context 訓練會傷長輸入。
- Jev agent harness 實測（jev-1.13.0，約 22,500 calls）：局部可觀察的判斷有效（injection、rerank、routing、shell risk gate）；預測其他模型會不會失敗（RouterBench 51.3%）與跨步因果歸因（AUROC 0.560）失敗。教訓：把模糊判斷拆成正交的 yes/no，再由程式組合；低分不代表安全。JEVal 的 τ-bench：Jev＋LLM 成功率 77.6%→66.1%，時間 267.1→160.6 s。
- 2026-10-22 給使用者的建議（未實驗）：Luna low 應定位為 teacher／上限，不是「假裝的 decision model」；RTL 決策的標籤可由 solver 結果自動產生，可用來 fine-tune（kettei 支援 fine-tune）或走 LLM-as-Jev 路線，量化 decision model 與 reasoning LLM 的差距。

## 他人如何主張 decision model 比 LLM 有用

- Edge orchestration（arXiv `2609.22753v2`）：下游 validator／policy 固定，只換 interpreter；LLM 用相同欄位的嚴格 JSON、關 reasoning；另設同權重 JSON 版以隔離 readout 效果；預註冊假設、bootstrap、McNemar、Holm。結論只限 bounded contracts：4 欄 exact match 0.923–0.967 對 DeepSeek 約 0.973–0.983，每次正確決策費用低 59.7–80.9%；8 欄跌到 0.527–0.577（DeepSeek 0.900–0.930）。
- Rubric judges（arXiv `2609.29769v3`）：人類標註 9 panels；decision model 最多在 6/27 比較中顯著落後 LLM，成本低 16–357×；兩者錯在同一處，以 confidence 轉交 LLM 的 cascade 最多只多 2.7 點。
- Pentest harness（arXiv `2609.28940`）：decision layer 只能降級或標記，不能推翻 deterministic validator；但每組只跑一次，屬弱證據示範。
- 不計時間時，速度／成本的主張失效；剩下可主張的是：機率排序能減少昂貴 solver 嘗試、低價可對每個訊號打分、能用 solver 自動標籤 fine-tune。

## 呼叫 GPT-6 Luna 的路徑（2026-10-22 實測）

- mazu 的 local gateway `/v1/models` 列出 `gpt-6-luna`（上游 opencode Go，官方頁也列入 Go 的 $10/月方案），但 `/v1/chat/completions` 回 HTTP 400 `model_not_allowed`，allowlist 只有 `deepseek-flash`、`deepseek-pro`、`deepseek-v4-flash`、`deepseek-v4-pro`。要用須修改 gateway 設定，未改動。reasoning effort 是否被 gateway／opencode Go 保留也未驗證。
- **可用路徑（2026-10-22 實測 HTTP 200）**：直接打 opencode Go `POST https://opencode.ai/zen/go/v1/responses`，帶 `x-opencode-session` header（缺少時回 `MissingSessionID`），body 用 `reasoning.effort=low`；回應顯示 effort low，回報 reasoning tokens。同一模型走 `/chat/completions` 回 `ModelProtocolUnsupported`，`/v1/decisions` 回 404。
- NeuroAbs 公開程式碼（`ZhiyuanYan/NeuroAbs`，使用者 fork 於 `swear01/NeuroAbs`）只有 per-statement rewrite 用 LLM；候選訊號由 `*_constant.v` 加 Yosys constant propagation 的 OPT_CLEAN 差異決定，不是 LLM signal analyzer。先前 Muse I2C 每次 call 的中位數為 4,610 prompt／2,845 completion tokens，用 Luna 牌價估約 $0.002／call。
- OpenAI 直連 Luna $0.10／$0.50 per M；Batch 或 Flex 半價 $0.05／$0.25（OpenRouter 也有 `:batch`／Flex 同價）。研究生成不可走 opencode／Codex CLI 的內建 system prompt，應直接呼叫 API。

## Datapath UF、memory abstraction 與 NeuroAbs 的決策點

- 使用者決定不改 IC3 底層（generalization、clause 等）；決策只放在 RTL／模型層。
- UF 與 memory abstraction 不是只能人工：Reveal（Andraus／Liffiton／Sakallah）以 bit width 判定 datapath，自動換成 UF／UP，再以 CEX＋MUS refinement；AVR（TACAS 2020、HWMCC 2020 冠軍，也是 NeuroAbs 的 backend）內建 syntax-guided／equality abstraction。Memory 有 Bjesse／Synopsys 專利 US8001498B2（CEX 模擬找錯誤讀值，再加 abstraction pair）、Manolios ICCAD 2006、Charvát 等的有限次存取模型。這些用固定 heuristic 選初始抽象，再以 CEGAR 修正；LLM／decision model 可提供初始選擇與 refinement 優先序，baseline 須含 AVR 的自動抽象。
- NeuroAbs 決策點：LLM 決定哪些訊號與 scenario 相關、localization 時固定哪些控制值；AST traversal 決定要改哪些 statement；LLM（gpt-4o-mini，每 statement 約 3.70 s）決定子運算式改 X、化簡或保留；Z3 檢查 overapproximation；R-CEGAR 決定要還原哪些 statement。
- NeuroAbs 公開 xlsx（Figshare v2，仍無程式碼）：AVR 多數加速，rIC3 幾乎持平；有變慢案例（Piccolo SUB AVR 1096.61→1450.38 s；PicoRV32 SUB 663.77→763.36 s）。PicoRV32 riscv-formal-ADD：AVR 862.77→561.66 s、rIC3 39.32→36.1 s，與本專案 PicoRV32 ADD pilot 是同一題。
- AIsimpV 的 9/21 實驗只跑 NeuroAbs-inspired netlist cutting（三個 SFIFO，12 次全選空集合）。另在 NeuroAbs fork 已用 Muse 1.2 完成 I2C 單題重現：184 次改寫皆 sound，rIC3 UNSAT，但 0.105 s 對具體 0.100 s，無加速；PicoRV32／Piccolo 尚未重現。

## 對本專案的含意

- 既有 bundle 很大（PicoRV32 majority prompt 84,726 tokens），整份 RTL 直接餵 decision model 會超過 20K 上限且遠超 500 ms；必須先由 Yosys／COI 抽成每個選項的短特徵（訊號寬度、fanin/fanout、與 assertion 距離、CEX 摘要）。
- 每次 solver 呼叫是秒級（0.3–60 s），單次決策 500 ms 不是瓶頸；延遲只在「每個訊號／每個 proof-tree node 都要決策」的大量呼叫時才重要。
- Decision model 只能替換「選擇」，不能產生 abstract RTL、h/J/w 證書或 invariant；soundness 仍由 formal gate 判定。
- 舊 SFIFO 的單點 havoc 空間 33/33 全為 CEX（見 [checker 記憶](certificate-checker-v0.md)），在那個空間換更好的選擇器也不會成功；測 cutpoint 選擇需改用確實存在可行 cut 的題目。
