---
title: "Routine 論文細項評分缺漏與補評"
scope: projects/routine
status: active
updated: 2026-11-10
---

- 日報有論文總分但缺星等時，先讀 GitHub Actions 的 `daily-report` artifact，分開四細項缺漏、有效空選擇、來源／模型失敗及 Discord 送出。上次未留 raw reply，缺 key 或格式不符未隔離。
- [PR #13](https://github.com/swear01/routine/pull/13) 要選文模型產四個細項，只補評入選文章的缺漏一次，保持排名、總分及現有細項。仍缺顯示「未提供」。來源成功且選擇經驗證為空時保持靜默。空陣列本身不能證明來源查詢完整，也不能吞掉 timeout／限流。
- Dry run 核 artifact 的排序、總分和細項，分開驗證正式 Discord delivery。當次只驗 dry run 與 CI，正式送達未測。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/routine/paper-rating-completeness.md)

合併自 [空選擇的舊調查](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/routine/validated-empty-selection.md)。
