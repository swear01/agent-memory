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
- `/api/sessions` 回 1226 筆（1190 archived）~1MB，gzip 115KB、JSON.parse ~4ms，只在 invalidation 時 refetch（平常走 SSE patch）；groupSessionsByDirectory 等處理 ~26ms/60s。不值得改（要改就是 hub archived 分頁，屬功能改動）。
- 2026-10-25 第二輪 A/B（拔掉 #2061 後、只量閒置 60s、加 long-animation-frame）：基線 1207/1043ms → layout effect 改成只跑一次（上限估計）922/992ms，省 ~5–25%，接近雜訊；render 階段 layout 僅 ~10ms。#2040 的 no-deps `useLayoutEffect` 不值得冒 deps 漏寫的版面 bug 去改。#2040 是 heavygee 的、已 merge upstream；#2061 是 techotaku39 的、仍 open。
- 方法：正式站沒 sourcemap；`vite build --minify false --sourcemap` + 小 Bun proxy（要刪 `content-encoding`）即可對真資料 profile。
- 2026-10-25 修法：上游 PR tiann/hapi#2165（branch `fix/web-sidebar-rerender`，最新 head `8cdde739`，HAPI Bot 無 finding、CI 綠，等待維護者合併）。內容：`memo(SessionItem)` + router `onSelect` useCallback + `CollapsiblePanel`（收合 transitionend 後卸載列）。HAPI Bot 在前兩版抓到三個 memo/卸載帶來的真回歸，都已修：(1) 相對時間原本靠整個列表重繪順便更新，memo 後會停住 → `SessionRowSummary` 自己 `useMinuteTick()`；(2) Codex 匯入標記原本由 SessionList 訂閱、只為了強迫全表重繪 → 改在 `SessionRowSummary` 用 `useSyncExternalStore(subscribeCodexImportedSessions, …)`，並刪掉列表層訂閱；(3) 群組在最後一個 active session 斷線時會自動收合，卸載會丟掉開著的 Rename/Export 對話框 → panel 透過 context 提供 hold，列上有選單/對話框開著就不卸載。機器 heartbeat「略過 activeAt/health」不成立：health 會顯示在機器列。A/B 閒置 60s：fork v0.30.7.6 4.0s→0（即使保留 #2061）；upstream main 391ms→0；DOM 4.9k→1.5k。
- 已在 tiann/hapi#2061 留言（comment 6401125991）說明 per-row `useIsMutating` 的 O(rows²) 通知，建議把訂閱移進 `DeleteSessionDialog`（只在開啟時存在），並連到 #2165。
- 已隨 fork v0.30.7.7 發布並部署（#2165 carry）。注意：#2165 讓收合群組不掛載列，carry 的 #1418／#1944／#2080 測試原本直接點收合群組裡的列，要先展開群組（v0.30.7.7 用 overlay 修測試）。
