---
title: "Routine 論文細項評分缺漏與補評"
scope: projects/routine
status: active
updated: 2026-11-10
---

- `swear01/routine` 的 2026-11-10 排程 [42757735741](https://github.com/swear01/routine/actions/runs/42757735741) 成功，報告有三篇總分，12 個細項只顯示 1 個。先查 `daily-report` artifact，區分評分缺漏、有效空選擇、模型呼叫失敗及 Discord 發送失敗。原始模型回覆沒有保存，無法確定是缺 key 還是格式不符。
- [PR #13](https://github.com/swear01/routine/pull/13) 合併為 `a6c6a1d`。選文模型須提供四個細項；只針對入選論文的缺漏做一次補評，保留排名、總分及有效細項。仍缺的欄位固定顯示「未提供」。無入選時仍靜默結束。
- 41 項測試、主分支 CI 與最新提交 Gemini review 已驗證。分支 [dry run 42770358557](https://github.com/swear01/routine/actions/runs/42770358557) 以初版修復提交 `f190a9a` 執行成功，報告三篇各有四項星等，沒有「未提供」。未測正式 Discord 交付，未更新 Gist，也未重發舊訊息。
