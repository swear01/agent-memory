---
title: AIsimpV RTL certificate checker 的已驗證界線
scope: project
project: AIsimpV
status: active
updated: 2026-10-21
---

本線入口為 [AIsimpV](https://github.com/swear01/AIsimpV) 的 `rtl_relate` 與 [驗證問題分解](certified-problem-decomposition.md)。既有 small certificate 和 native product 的適用範圍分開；舊 proof-lifting 設計處於暫緩。

## Soundness 與真實 RTL

- **Clock 名稱或沿不同仍被接受。** Frontend 保存驗證過的 clock name/edge 並與 frozen contract 精確比對。五個基本義務共用同一 witness w，用 STEP_MAP 要 z=0、OBS_MAP 要 z=1 的 Mealy 控制測共同 witness；還要檢查 STEP_J 與非空 initial set，避免空泛通過。每拍 fresh z，property 不注入 h/J/w。
- **BMC 沒有反例，或 exact replay prefix SAT。** 單獨 BMC 只給 BOUNDED，公開 RTL SAFE 需 base 和 k-induction；tiny 模型經完整 reachable closure 或一步完備證明才可 SAFE，碰上有限探索上界回 UNKNOWN。Prefix SAT 要核 frozen property 真正違反才是 BUG；另外找到的 concrete trace 不得合併成原 trace 的 concretization。
- **Yosys Witness 重建後不對，或反例深度誤寫成上限。** Witness `bits` 字串反向對應 LSB-first signal IDs；缺少 input bit 時明記驅動為零並重新驗證。用真實輸入與 typed model 還原後，重查 A-prefix SAT 及 frozen P 確實違反，再 exact replay C。反例保留實際 failure step，SMTBMC 的 `requested_depth` 是請求上限。
- **Frontend 接通但上游測試條件被悄悄改寫。** AVR `h_Arbiter` 的請求在原始配置永不發生，改成自由 input 是另一變體。Pipeline `$check` 只走固定 hash、保留監看器且驗證 frozen predicate 等價的 extraction。ZipCPU skidbuffer 未編 `FORMAL` 的衍生 property 不是完整上游套件。舊 sfifo `$memwr_v2` 在有限 checker unsupported，不能把未初始化 memory 補零。
- **Native product SAFE 卻限制了原始 public input。** 只允許實際 stored state 的非平凡 init，拒 initializer alias／memory read-register 衝突與 async read reset。禁止額外 assumptions，原 A/EN、任意初始 memory、共享原 anyconst、每拍自由的新 abstract inputs、所有 public outputs 和 enabled violations 都需保留。Helpers 是 assertions。這套 universality 條件提供充分證明，也會拒掉某些合理摘要。
- **外層 timeout 留下 solver descendants。** 傳共同 deadline 到每個內外 worker，內層先清理自己的 process group，再讓外層結束。以真實 sleeping-child fixture 驗沒有活著的 solver；rejection、exception、fallback、cleanup 都算成本。重跑用 `python3 -m unittest discover -s tests -v` 及 `python3 -m rtl_relate demo --out results/<new-run>`，保留 raw logs、SMT、RTL、traces 和 hashes。CI 透過 tee 時明確 shell bash／pipefail。

## 解讀研究結果

- **Property proof 約 0.3 s 就寫成百倍加速。** 三個 SFIFO 約 30 秒案例是同家族的不同配置。比較匹配過 normalization 的 C；直驗 safety theorem 若被轉移到 certificate，仍屬正確性成本。已測 3 組的 property 約 0.3 s，但含 certificate 的配對總驗證只有 1.09×／1.21×／0.92×；生成約 573–901 s 另列。AI 有自主提出 21-bit summary 的案例，驗證正確仍慢於 matched C。保持全部失敗、重複證明與候選來源。
- **模板或資料不全時說已比較其他研究。** 本輪 NeuroAbs artifact v2 只有結果表，僅做明標 inspired 的 netlist cut；沒有原始 AST rewriting／CEX reduction 或 AVR 求解對照。33 個單點 havoc 皆假反例只縮小本次 monotone 搜尋空間。使用者要求直接讓 AI 搜尋約三個 baseline 30 s 的案例，另外比傳統模板，不要求先有人造成功摘要或全數超時。硬體 safety 與 PPA/equivalence 的基準也不能互換。
- **正式生成誤走 Codex CLI 或官方 DeepSeek 帳戶。** `scripts/llm_pilot.py` direct API 每請求僅完整 user message，連 model config 一起凍結。DeepSeek 走既有 local gateway，Muse 走既有 Meta 路徑。保存 client exact body、returned model、usage、固定 `x-opencode-session`、HTTP status 與 gateway attempt/endpoint。Gateway 可能更動實際上游 bytes；官方 402 和 gateway Cloudflare 1010 屬不同路徑。只有成功 transport smoke 不能支持 RTL discovery。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/rtl-relate/certificate-checker-v0.md)
