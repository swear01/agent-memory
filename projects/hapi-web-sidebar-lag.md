---
title: HAPI web 側欄卡頓：#2061 每列 useIsMutating 造成 O(rows²) mutation-cache 風暴
scope: projects/hapi-web-sidebar-lag
project: hapi
status: active
confidence: high
evidence: v0.30.7.6 headless Chromium A/B、long tasks 及 CPU profile；2026-11-28 GitHub 讀回 #2061、#2165 均合併。
created: 2026-10-25
updated: 2026-11-28
tags:
  - hapi
  - web
  - performance
---

- 側欄閒置長卡先看 sidebar 是否每列訂閱 `useIsMutating`。TanStack v5 每列的 `useMutation` 若用新 closure，`MutationObserver.setOptions` 會通知各列搜尋／深度比對 mutation cache，隨列數平方放大。單次 A/B 移除 per-row 訂閱大幅降低 blocking。把訂閱放到實際開啟的 `DeleteSessionDialog`，另檢 machine heartbeat `upsertMachine` 造成的全列表重畫；機器 health 是可見欄位。
- 收合只做 CSS `grid-template-rows: 0fr` 仍會掛載所有列。[上游 #2165](https://github.com/tiann/hapi/pull/2165) 用 `memo(SessionItem)`、穩定 onSelect、`CollapsiblePanel` 動畫後卸載。memo 後讓 `SessionRowSummary` 自己跑 `useMinuteTick()`，以 `useSyncExternalStore` 訂閱 Codex import。Keep-mounted hold 保留群組自動收合時開啟的 Rename/Export dialog，測試先展開群組再找列。[#2061](https://github.com/tiann/hapi/pull/2061) 和 #2165 已合併，查看站點實際部署再判定。
- 正式 build 無 sourcemap 時，以 `vite build --minify false --sourcemap` 搭 Bun proxy 接同資料（移除 `content-encoding`）。用 long task/animation frame profile 比較同條件。舊樣本的 sessions JSON parse 與 layout effect 影響較小，有新 profile 才決定是否動分頁或 layout。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/hapi-web-sidebar-lag.md)
