---
title: HAPI web 側欄卡頓：#2061 每列 useIsMutating 造成 O(rows²) mutation-cache 風暴
scope: projects/hapi-web-sidebar-lag
project: hapi
status: active
confidence: high
evidence: 2026-10-25 在 oracle 用 headless Chromium 對正式站 v0.30.7.6 量 long task + CPU profile；用同版本未 minify build（proxy 到正式 API）做 A/B。
created: 2026-10-25
updated: 2026-10-25
tags:
  - hapi
  - web
  - performance
---

- 現象：v0.30.7.6 閒置時主執行緒每 ~10s 卡 300–1100ms，載入時一次 2.5s；60s 內 long task 合計 ~9s。
- 主因：fork carry 的 upstream PR #2061（未 merge）在 `useSessionActions` 加 `useIsMutating`，每個 SessionItem 一個 mutation-cache 訂閱。每列另有 15 個 `useMutation`；TanStack v5 `MutationObserver.setOptions` 在 options 非 shallow-equal（inline mutationFn 每次 render 都是新 closure）時會 `mutationCache.notify`，所以每次列表 render = rows × 15 × rows 次 `findAll`+`replaceEqualDeep`。284 列 ≈ 120 萬次。
- A/B：只把那一行改成 `false` → 60s blocking 9.0s → 2.7s（兩輪一致）。
- 觸發源：machine heartbeat `machine-updated` 送完整 record（只有 activeAt/health 變），web `upsertMachine` 永遠換新物件 → SessionList 全量 re-render。session keep-alive 已被 `isRenderIrrelevantPatch` 擋掉，machine 沒有。
- 放大器：收合的 group 只是 CSS `grid-template-rows: 0fr`，子元素仍掛載（284 列中 202 列看不見）；SessionItem 沒 memo；SessionList 有個無 deps 的 `useLayoutEffect`（#2040 toolbar shed）每次 render 強制 layout。
- `/api/sessions` 回 1226 筆（1190 archived）~950KB，非主因。
- 方法：正式站沒 sourcemap；`vite build --minify false --sourcemap` + 小 Bun proxy（要刪 `content-encoding`）即可對真資料 profile。
