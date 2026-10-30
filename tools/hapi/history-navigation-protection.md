---
title: HAPI history navigation leases, transcript gaps, and browsing handoff
scope: tools/hapi
project: hapi
status: verified
updated: 2026-10-30
source_refs:
  - github:tiann/hapi#1597
  - github:tiann/hapi#1542
---

## 整合時需保留的三種狀態

#1597 回覆導航與 #1542 串流 history protection 不是單純解衝突。Conversation-start jump 保留 bounded head/tail；outline 與 response-to-input jump 必須保護中間目標；導航結束後仍在 history browsing，保護也不能立即消失。

- `setMessageViewMode('tail')` 的 navigation lease guard 必須先於同模式重設／boundary release，否則 near-bottom frame 會取消尚在載入的導航。
- 只保留最舊 600／最新 400 筆會移除超過 1,000 筆載入範圍的中間目標。Outline 與 response-to-input 共用 history-preserving lease；conversation-start 仍走 bounded window。
- Release target lease 時，若仍為 history mode 且缺少 browsing boundary，建立 boundary 來接續保護。否則 conversation-start jump 後選取 retained tail，再於 lease=0 後收到 SSE，普通 history trim 會移除剛選取的輸入。回到 live tail 才解除 boundary。
- Leases 必須 reference-counted、release idempotent，teardown 不得提前釋放其他導航的保護。
- 合併 queued message、agent-run event 或替換 reasoning snapshot 時，regular row count 不一定增加。不能先移除 gap marker、只在超過 cap 時重建；資料仍缺頁卻沒有警告，會讓 reply attribution／分享跨錯誤 prompt。既存 gap 不占 regular row budget；保留區段的 gap 也要保留。

## 驗證方法

以真實 store 與 HappyThread 測試：先載入 1,200 筆，從 outline 和 response footer 選中間目標，要求至少一筆 streaming ingest 發生在 navigation lease > 0 期間。另測 conversation-start → retained-tail input → lease=0 → 再 ingest，目標仍可見。

Heavy DOM fixture 連續 20 次／秒 ingest 曾使 Linux CI 超過 90 秒。導航 race 用有限三筆更新並明確檢查 lease overlap；另外保留持續 streaming 的獨立測試。等待 Load earlier enabled 與精確 loaded count，不以固定 sleep 或跳過 assertion 掩蓋失敗。Response footer 不可透過 DOM click 點擊 outline overlay 後方的不可操作介面；先正常完成 outline 導航與關閉再測。

## 測試與架構防護避坑

- **Context 屬性可選化防 Mock 污染**：`HappyChatContextValue`（`web/src/components/AssistantChat/context.tsx`）擴充導航狀態（如 `conversationStartStatus`、`promptNavigationStatus`、`jumpToMessage`、`abortNavigation`）時，必須設為可選（`?:`）。若宣告為必填，會引發 shotgun mock 污染，逼迫超過 8 個與導航完全無關的元件測試（例如 `CodexPlanFooter.test.tsx`、`ToolGroupCard.test.tsx`、`markdown-a.test.tsx`、`ToolMessage.generatedMedia.test.tsx` 等）補寫 mock，導致 PR 觸碰檔案暴增且製造無謂 diff。保持可選能讓改動集中在真正的導航元件上。
- **E2E 測試的大綱面板退場等待競態**：在 `e2e/load-earlier-running-session.spec.ts` 中，點擊大綱項目（`aside button`）觸發跳轉時，大綱面板（`aside`）需要非同步動畫退場。若未等待面板關閉就直接斷言目標進入可視區（`expect(selected).toBeInViewport()`），會因 overlay 遮蔽或版面尚未穩定而拋出 `viewport ratio 0`。點擊大綱按鈕後必須先等待 `await expect(page.locator('aside')).toHaveCount(0)`，再做可視區斷言。

2026-09-14 證據：#1597 `17972547` 整合 #1542 `f8953d24`；root typecheck、3,187 web tests、最新 head CI／review 通過。Store handoff regression 已先重現失敗，再驗證修復；21 項 browser gate 與擴充後的 retained-tail regression 有實測。當時 PR 仍未合併／部署；之後須重查 live head，整合順序為 #1542 再 #1597。

2026-10-30 證據：#1597 在 `upstream/main`（`f8e55fb93`）基礎上壓合為單一 commit `ade45ad36`；移除 6 個無關 mock 檔後觸碰檔案收斂至 32 檔。修復 E2E `aside` 退場等待後，`bun run typecheck:web` 通過（0 errors），151 個 web 測試與 Playwright E2E 全綠。Codex PR Review 與 GitHub Actions CI 觸發驗證中。
