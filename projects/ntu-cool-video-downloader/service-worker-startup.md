---
title: NTU COOL 批量選單消失：service worker 頂層 await 啟動失敗
scope: projects/ntu-cool-video-downloader
status: active
updated: 2026-09-15
---

## 根因與修正

Canonical repository 為 `swear01/NTU-COOL-video-downloader`；legacy upstream 不作為修正基底。

背景模組 `background/background.js` 的頂層 `await restorePendingFilenames()` 會讓 Chromium 拒絕啟動 service worker。連帶無法執行 `runtime.onInstalled` 的 context menu 註冊，也無法處理下載訊息。右鍵選單缺少 `open-batch` 時，先驗證背景服務是否真的載入，不要只新增 UI 入口便宣稱根因已解決。

修正是先同步註冊喚醒事件，再呼叫 `void restorePendingFilenames()`。恢復函式本身已有錯誤處理；不要移除非同步恢復，也不要為了啟動修正把 `onDeterminingFilename` 改成永久監聽，否則會干擾其他下載器的檔名。

## 真實 Brave 單一變因驗證

Brave 152.1.94.117，使用獨立暫存 profile 和程式副本，呼叫真實 chrome API：

- 原 1.2.2：沒有背景 worker target；狀態訊息在測試的 1500ms 期限內未回應；`contextMenus.update('open-batch', {}, callback)` 的 `runtime.lastError` 為 `Cannot find menu item with id open-batch`。
- 同一份 1.2.2，只把上述頂層 `await` 改為 `void`：背景 worker 存在，`getStatus` 回傳 `{found:false, job:null}`，既有選單的 update 成功。
- 修正版 1.2.3 同樣通過。選單存在的 API 證據與獨立環境實測，不代表已在使用者原本 profile 重新載入或重跑批次。

## Node 測試的盲點

非同步 `import()` 允許頂層 await，mock chrome API 的單元測試因此曾全綠。新增以 `createRequire(import.meta.url)` 同步載入背景模組的回歸測試；舊版出現 `ERR_REQUIRE_ASYNC_MODULE`，修正後通過，並可捕捉 imported module 引入的頂層 await。同步載入檢查補上此模組限制，仍不取代真實瀏覽器測試。

延遲 storage 的測試應可明確控制 Promise 完成：模組與喚醒 listener 必須先註冊，之後才 resolve 恢復作業；不要用模糊的固定 sleep 或等待模組頂層 await 來證明啟動正確。

## 版本與操作界線

- 問題源自 commit `10f855802e0be5fe78f32ab8f9bdcfdb995ab92d`，1.2.1 原始碼已包含；不能把使用者重新載入後才看見失敗，直接當作 1.2.2 才引入的證據。先前實際快取版本未讀回，不能斷言。
- PR #16 補了 popup 原生批量頁連結，這是改善入口可見性；真正背景啟動修正是 PR #17、1.2.3。
- PR #17 受審 head `03ec5205b5ab875caef3584c2387f4fab281742a`；Swear Review 零 findings，79 項測試、CI、CodeQL、ClamAV 通過。合併 commit `245355771cbad1b80d44df2736a94201c392d277`。
- 本次本機檔案與 ZIP 已比對一致；沒有發布商店或 tag，沒有重跑原批次。來源碼、ZIP、安裝中版本、已生效背景 worker 是不同驗證層次；更新檔案後仍須重新載入並讀回版本。
- macOS 暫存 extension 路徑應先解析 realpath，再計算測試 extension ID；否則可能用錯 ID，看到 `ERR_BLOCKED_BY_CLIENT`。此錯誤不是 extension 啟動根因的證據。
- 工具封鎖實際管理頁時，不得改用替代介面存取相同受限目標。獨立測試 profile 只用自己的副本與資料，結束後清理自己啟動的程序。
