---
title: BCH native Opus 5.5 high 的 no-skill 修復版與 skills 實驗
scope: projects/spec2rtl
status: verified
updated: 2026-10-15
---

# 已完成的比較

Issue #35 原文是 Sonnet 5.5／Grok 4.7；以下是另外執行的 Claude Code 2.1.292、`claude-opus-5-5`、effort `high` 結果。先前執行 68.5 分鐘後中斷的 Opus 快照（cost 357,014.3146）僅為歷史資料，不得當作這兩輪的最終結果。

| 獨立驗收項目 | No additional skills，回報診斷後修復 | Pinned skills＋Codex supervision |
| --- | ---: | ---: |
| Clock | 2.00 ns | 2.00 ns |
| Area（µm²） | 59,385.312922 | 59,906.499301 |
| RTL / SDF gate patterns | 6/6 / 6/6 PASS | 6/6 / 6/6 PASS |
| Gate timing-violation messages | 0 | 0 |
| Setup / hold WNS（ns） | +0.000045 / +0.010901 | +0.000082 / +0.010495 |
| Σ(Time/n) | 5.761013 | 6.079681 |
| Cost = Area × Σ(Time/n) | 342,119.54 | 364,212.41 |
| pat0–pat5 gate times（ns） | 105, 225, 753, 75, 171, 629 | 117, 237, 753, 79, 175, 631 |

Cost 越低越好。Skills 版比 no-skill 修復版高 **6.46%**；相對原 issue 的 Sonnet `no-harness (2)`（cost 534,644），兩者分別低約 **36.01% / 31.88%**。Sonnet 沒有重跑，這是單次產物比較，不足以推論模型／skills 的平均優劣。

# No-skill 的 X 與修復

初始 prompt 僅兩行，未給架構、既有 RTL 或外加 skills／system prompt／hooks：

```text
Implement the BCH decoder in spec/spec.md as synthesizable SystemVerilog.
Goal: minimize mapped cell area × Σ(transaction time / code length), subject to correctness and spec/timing.sdc.
```

原生 agent 在 108 分鐘自行結束；獨立 RTL 6/6 與 STA 通過，固定 mapping 的 SDF gate 卻 0/6，出現 X／timeout。Agent 自己的 `compile_ultra` netlist 通過，不能代替固定 `compile -map_effort medium`＋incremental mapping 的驗收。

追到未 reset 的 syndrome state 在啟動時傳播 X；不用 SDF／timing checks 亦可重現，不能歸因為單純時序檢查。使用者授權回報診斷後，由 Claude 自己補同步 reset 與 testbench X/Z checks，沒有手改 RTL。修復約 11.8 分鐘，改動與自測後撞訂閱額度，未完成文字交付；freeze 後獨立驗收一次才得到表中通過結果。最初失敗版仍保留，不能將修復版描述成一次成功或無診斷協助。

# Skills 版的輸入與監督邊界

初始 prompt：

```text
Run the orchestrator on spec/spec.md in oracle mode.
Goal: minimize mapped cell area × Σ(transaction time / code length), subject to correctness and spec/timing.sdc.
```

- Skills 固定 `develop` commit `322f36c6fa9644e87d5b0f944bd1e28a79930136`，11 個 stage、43 個檔案前後 byte-identical；未修改 skills。
- 乾淨 workspace 起始只有 spec、SDC、pinned skills 與 git boundary，PPA 階段加入公開 synthesis configuration；未供給舊 RTL、分數、X/reset 診斷或 golden answers。
- Codex 只按 spec 回答 oracle 問題、核准 Claude 建議的 default architecture；記為 `Source: Codex operator`，不是人類核准。RTL 由 Claude 寫；沒有獨立 grader 的失敗回饋、手改或重抽直到成功。
- 正式 `Skill` invocation 只有 orchestrator 一次，另外 9 個 stage 的 SKILL.md 透過 Bash 閱讀；optional ArchView skipped。不要把 11 個可用 skills 說成 11 次正式呼叫。
- Native permission mode 與 no-skill 不同（`--dangerously-skip-permissions` vs `auto`）。加上 Codex 監督及前輪的診斷修復，這不是單變量 skills ablation；Cursor Sonnet 與 Claude Code 的 native harness 也不同。
- Native 自評使用 DC 2026.03，其 score 約 192,498 不是固定 grader 分數。表中數值均使用相同 benchmark、DC 2022.12-SP6、library、SDC 與 grader，在流程交付後 freeze、獨立評分一次。

Skills 版 native active time **2,695.016 秒（44.9 分鐘）**；output **216,330 tokens**，其中 thinking **126,234** 已含在 output，不能相加。三次啟動都沿用同一 native session，兩次 resume 是 oracle／架構 workflow gates；**quota wait = 0**。這輪未實際撞五小時上限，僅 controller recovery fixtures 通過，不能聲稱真實五小時恢復已驗證。訂閱 CLI 的 API-equivalent estimate 亦不是實際扣款。計數與隔離細節見 [native CLI 筆記](../../tools/claude-code/native-isolation-stream-usage.md)。

# 評分與續跑的防錯

固定 pat0–pat5 code lengths 為 **63, 255, 1023, 63, 255, 1023**。TB 的 `N` 是 transaction/frame count，不能拿來當 BCH code length n。Collector 曾因此誤算 cost 86,175,499.24448851；已在發布前修正 parser 與 metadata，以相同 frozen RTL／測試日誌重算為 **364,212.4092362056**，未重跑推論或驗收。Superseded reports 與 cost-correction evidence 保留；未來加 assertion 驗證 code-length mapping 與手算 cost。

使用者要求長實驗在 **Mazu** 跑，可用 HAPI Codex 操控原生 Claude，本機可能關機。若撞五小時額度，等 reset 再 resume 同一 Claude session，不重開實驗。這次用 Mazu user systemd＋linger 保持執行，完成後服務停止／disabled、HAPI session archived；不用因看到 archived session 就重跑。臨時登入資料已移除，原帳號登入資料保持不變；不得保存秘密於記憶。

# Provenance 與資料位置

- Benchmark：`ed2c429f5cd8a36876081fb0b0b9769935994c3f`；兩輪 independent grader 與 skills pin：`322f36c6fa9644e87d5b0f944bd1e28a79930136`。
- No-skill revised RTL SHA-256：`6aaf9f3736dd8d12bd3866aec38f30cb25ba28f40d7515418ca85dbfc9db12f0`。
- Skills frozen RTL SHA-256：`cdaa5723b057ca25848b1f19c9746dc37019e85ddaf4558ddf461ea1043a7a12`。
- Mazu evidence root：`<remote-home>/.cache/spec2rtl-opus55-skills-20261015-controller`。保留 `summary.json`、`summary.md`、`logs/`、`checkpoints/`、`evaluation/assessment.json`、`evaluation/frozen-output/bch/rtl/bch.v`、`evaluation/final-workspace.tar.gz`、independent netlist/SDF/logs 及 `evidence/cost-correction.json`、`evidence/caller-verification.json`。
- Skills workspace：Mazu `<worktree-root>/spec2rtl/bch-opus55-high-skills-20261015`；不要把獨立 golden grading 放進 agent workspace。
- 2026-10-15T14:57:41Z 已編輯 Issue #35 的 [原留言](https://github.com/DVLab-NTU/spec2rtl/issues/35#issuecomment-6193091516)，保留 no-skill 修復紀錄並附加 skills 結果，未新增留言；API 回讀與 body byte-for-byte 相符。Body SHA-256：`ef8b35b9c5e6fe0b83d493e29f449eeed232aaf7e1461c592c14cbf565d64f7c`。Mazu `evidence/issue35-comment-update-20261015T145741Z/` 保存 before、payload、PATCH response、readback verification。
