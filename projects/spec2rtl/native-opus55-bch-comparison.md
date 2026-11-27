---
title: BCH native Opus 5.5 high 的 no-skill 修復版與 skills 實驗
scope: projects/spec2rtl
status: verified
updated: 2026-10-16
---

- **Agent 自測與 STA 通過，固定 grader 的 gate sim 卻全 X。** 使用同一 frozen RTL、benchmark、library、SDC 與固定 DC 2022.12-SP6 mapping（`compile -map_effort medium`＋incremental）獨立重驗；agent 的 `compile_ultra` netlist 不能代替。No-skill 初版未 reset syndrome state，無 SDF／timing checks 也能重現。回報診斷後由原 agent 補同步 reset 和 X/Z checks；原失敗保留，修後通過不能回寫成首輪成功。
- **用 TB 的 N 算 `Area × Σ(Time/n)`。** N 是 transaction/frame 數，pat0–pat5 的 n 固定 `63, 255, 1023, 63, 255, 1023`。在 collector 以 assertion 與手算核對；誤算改原始日志的衍生計分，不重跑推論。
- **有技能的一輪被拿來說平均更好／更差。** 兩臂的額外 skills、permission mode、oracle 監督與前輪診斷協助不同。記 skills pin、workspace 輸入、真正 Skill invocation 和 Bash 讀文件；Codex operator 回答 spec／批准架構與人類批准分開。兩組是固定 grader 的單次產物比較，native 自評不混用。Thinking 包含於 output tokens；五小時續跑 fixture 不代表實際 quota wait/recovery。長實驗用 Mazu 原生同 session resume，別重開已交付 run。
- **Issue #35 原有比較表被長篇報告換掉。** 沿用原留言的 `Config | Model | Clock | Area (µm²) | sum Time/n | Cost | vs best` 及 gate-times 格式，新增兩列後按全表最低 cost 重算 vs best。保留 no-skill 診斷後修復與 skills Codex operator 協助等條件，精確讀回修改後原留言。[Issue #35 比較表](https://github.com/DVLab-NTU/spec2rtl/issues/35#issuecomment-6193091516)

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/spec2rtl/native-opus55-bch-comparison.md)
