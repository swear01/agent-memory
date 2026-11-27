---
title: "原生協定有方法不等於 HAPI 已接通"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: ab18c800cf25225331da4e6c2078f7529fe14ceda0d7e427acd6f494823f130e
---

# 原生協定有方法不等於 HAPI 已接通

歷史助手以安裝版產生的 app-server TypeScript 定義說明 turn/steer，再搜尋當時 HAPI Codex 路徑未找到接入；source 因此將協定能力與整合缺口分開。

用指定版本的生成定義核對 schema，再沿整合呼叫、進行中 turn 身份、拒絕與競爭條件驗證 runtime。沒有字串命中只是限定搜尋範圍的證據，不能當所有版本永遠沒有；自動 fallback 也不能造成重送。來源沒有 PoC 或實際 steer 成功，當時欄位不作現行 API 保證。

## ACP 標題、usage 與 steering 回報

- ACP title 事件分開核對官方 schema、各已安裝 client 的實作與原始事件。某次沒有 title event 或未安裝 client，只能記為未證實。是否加入 fallback 依任務授權；歷史案例使用者要求查官方資料且不要 fallback。
- Usage 順著原始 client 事件、轉換、保存與報表逐段驗證。只找到通用 receiver 不能說每個 client 都會送，沒有原始事件也不能指認特定 client 已修好。
- 搜到 binary 的 steer 字串不證明 SDK 可觸發，鄰近 stream-json 找不到也不證明只限 TUI。先查對外 schema 與實際 callback，保留未完成 runtime 驗證的界線。

來源回查：`git show 2505b5e:tools/hapi/client-event-capability-evidence.md`、`git show 2505b5e:tools/hapi/client-usage-event-proof.md`、`git show 2505b5e:tools/hapi/binary-token-reachability-proof.md`。
