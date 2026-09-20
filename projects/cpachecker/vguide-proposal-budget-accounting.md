---
title: VGuide 多次提案與失敗 round 必須按實際請求計算預算
scope: projects/cpachecker
status: active
updated: 2026-09-21
---

# 已驗證的預算缺陷與修正

PR #270 修正四個會污染 one-shot 與逐輪比較的 production 行為：

- 第一輪 SAFE 原本強制只抽一次，忽略明確設定的 `llmSamplesPerCall=K`。
  現在首輪與後續輪都尊重 K；預設 K=1 不變。
- Ensemble 原本把所有回答合併後套用單份 response 上限，第一份回答填滿時
  後面回答完全被丟棄。現在每份先按 head/formula 去重並限額，再跨回答穩定合併；
  重複候選不吃同一份回答的 unique-item 額度。
- 新增 `vguide.enableValidationFeedbackRepair`，預設 true；等預算實驗可在兩臂
  都設 false，避免 repair 額外消耗請求。這不是通用全域預算框架。
- scheduled/source-prior 的 primary IOException 現在仍扣 round 與 wall 額度。
  否則每個後續 refinement 都可能把失敗視為未使用額度而繼續呼叫。

Round 不等於 HTTP 次數：每輪 K 次 primary draws、retry 與 conditional repair
必須分開記錄。#269 固定比較為 first_spurious K=4 與 every_n K=1/max4 rounds，
兩臂 retries=0、repair=false；失敗與未觸發都保留，不補抽。

## 驗證與邊界

- PR: https://github.com/swear01/cpachecker/pull/270
- 接受 head: `6ac66f131985116b0c014324153c17bf2fe6d7de`；merge:
  `64d5888775a9d99c35c2a26a6dbb4aaccff9da8a`。
- Options/Ensemble/RepairFlow/Scheduler 共 33 個 focused tests 通過；包含真实 bridge
  六次 refinement 連續失敗只允許四次 request、首輪 K、候選合併與 repair 關閉。
- clean build 保留 Error Prone 與 -Werror，只沿用已知 IdentityHashMapUsage narrow
  exception；checkstyle 通過，exact-head Swear Review 零 findings。
- 報告：`<experiments-root>/reports/issue269-fixed-panel-20260921/`。

以上是排程、預算與候選處理修正，未證明新增解題。固定 development panel 的 live
verifier/provider 結果仍要另行驗證；不能以 33 tests 或 executable packet 代替收益。
