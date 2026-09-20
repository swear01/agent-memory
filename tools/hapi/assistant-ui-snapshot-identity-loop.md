---
title: "assistant-ui useSyncExternalStore 快取失效與 caret 依賴漂移死循環"
scope: tools/hapi
status: active
updated: 2026-09-21
---

# assistant-ui useSyncExternalStore 快取失效與 caret 依賴漂移死循環

## 現象與錯誤

在 HAPI Web 對話介面（特別是長對話歷史冷加載後）向上滑動時，畫面立即白屏崩潰並顯示 Error Boundary：
`Maximum update depth exceeded. The result of getSnapshot should be cached to avoid an infinite loop.`（Production Minified React error #185）。

## 根本原因

此錯誤由套件底層引用不穩定與依賴版本漂移共同造成：

1. **React 18 useSyncExternalStore 契約**：
   React 18 的 `useSyncExternalStore` 要求在外部狀態未變動時，`getSnapshot()` 必須維持引用同一性（`Object.is(inst.value, getSnapshot()) === true`），否則判定狀態永遠在變動而陷入渲染無窮迴圈（超過 50 次渲染上限）。

2. **@assistant-ui/core@0.2.23 訂閱與快取缺陷**：
   - 在 `ThreadListRuntimeImpl.subscribe(callback)` 中，錯誤地將回調綁定至 `this._core.subscribe(callback)`，而非內部的 `this._stateBinding.subscribe(callback)`。
   - 負責快取的 `LazyMemoizeSubject` 永遠沒有活躍訂閱者（`isConnected === false`）。
   - 在未連接狀態下，`LazyMemoizeSubject.getState()` 每次被調用皆重新配置一個全新的物件字面量 `{ mainThreadId: ... }`，導致記憶體指標每一次都不同。

3. **Caret 依賴漂移（Dependency Floating Skew）**：
   - `@assistant-ui/react@0.14.29` 宣告依賴 `"@assistant-ui/tap": "^0.9.6"`。
   - 官方 `v0.30.7` 發布時的預編譯 binary 鎖定在舊版 `tap@0.9.8`，當時 tap 尚未引入 PR #5897 的嚴格 loop guard，因此暫時未拋錯。
   - 當在維護分支重新執行安裝或更新 lockfile 時，tap 自動解析升級為 `tap@0.9.18`（`>= 0.9.14`），該版本啟用了嚴格的 snapshot 一致性防護，直接攔截未快取的 snapshot 並拋出崩潰。

4. **觸發源頭**：
   上游 PR #1809（Commit `b41b9293e`）將冷啟動訊息載入量縮小至 20 則，使用者稍微滑動畫面便會觸發頂部 coverage 動態載入，驅動 `SessionChat` / `useExternalStoreRuntime` 頻繁調用 `setAdapter`，進而引爆死循環。

## 解決方案

透過 `pnpm patch` 建立 `@assistant-ui/core`（0.2.23 版）持久化補丁：
1. 將 `ThreadListRuntimeImpl.subscribe` 修正為訂閱 `this._stateBinding`。
2. 在 `LazyMemoizeSubject.getState()` 加入 `shallowEqual` 淺比較快取，確保即使在未連接狀態下連續讀取，若資料實質未變則恆定回傳相同之物件引用。

## 驗證原則

此類狀態引用死循環無法被傳統靜態 mock 單元測試捕獲。驗證必須透過瀏覽器端真實 DOM 滾動或端對端測試（如 Playwright 在實際長對話連續快速捲動）驗證 0 page errors。
