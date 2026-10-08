---
title: NTU COOL 內建瀏覽器 filechooser 逾時改用 Enter
scope: tools/computer-use
status: verified
updated: 2026-10-08
tags:
  - ntu-cool
  - filechooser
  - browser
  - submission
---

# 問題與已驗證解法

2026-10-06 CVSD Lab2 與 2026-10-08 LSV Homework 1 都遇到相同現象：內建瀏覽器的 AX click、locator click 沒有開啟檔案選擇器，`waitForEvent('filechooser')` 因而逾時。檔案尚未選取，不能將此現象稱為上傳傳輸逾時。

COOL 上傳優先沿用已驗證的 Enter 方法。先讀取目前 AX，取得 file input 的最新 index，再建立 filechooser 等待並按 Return：

```js
await tab.ax.write();
// fileInputIndex must come from the current AX state.
const chooserPromise = tab.playwright.waitForEvent('filechooser', { timeoutMs: 15000 });
await tab.ax.pressKey(fileInputIndex, 'Return');
const chooser = await chooserPromise;
await chooser.setFiles([absolutePdfPath]);
await tab.ax.write();
```

確認頁面顯示正確檔名後，對目前 Submit Assignment 控制項按 Return。回執下載連結若 click 沒有反應，同樣先建立 `waitForEvent('download')`，再對當前下載連結 index 按 Return。每次操作前重讀狀態，不沿用舊 index。

本次 `DOM.setFileInputFiles` 被工具明確拒絕並要求使用 filechooser；不要重試該命令。顯示瀏覽器也沒有修復 click。只能確認 click 未觸發控制項而 Enter 有效，底層原因尚未查明。

# 授權與驗證

沿用已授權、已登入的內建瀏覽器；不讀取或搬移 cookies，也不自行改用使用者私人瀏覽器。登入與 CAPTCHA 留給使用者處理。

使用者已明確授權該次繳交時，完成準備與檢查後直接送出，不重問。這次授權不等於未來每次作業都可自動繳交。

完成證據必須包含正確課程與作業的 Submitted 回執、檔名與時間。能下載時，下載已提交檔案，以 SHA-256 和 byte comparison 核對本機原稿。兩次案例均有回執和下載逐位元相同的證據。作業清單可能顯示舊的 No submission；先查看詳情頁回執，避免重複送出。

# 共用記憶的位置

其他 agents 應透過 canonical `agent-memory` Markdown 與 QMD `memory` collection 取得此解法。只寫入 Codex 自動記憶或只更新本機 QMD，不構成跨 agent 同步。更新 canonical 筆記後完成檢查、Git push 與遠端核對，再刷新本機 QMD 並搜尋回讀。
