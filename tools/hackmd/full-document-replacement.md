---
title: HackMD 長文整篇更新與圖片驗證
scope: tools
status: active
confidence: high
created: 2026-09-03
updated: 2026-09-03
tags:
  - hackmd
  - codemirror
  - browser-automation
  - image-upload
sources:
  - live browser editing and verification
---

# HackMD 長文整篇更新與圖片驗證

## 已驗證的問題

- HackMD 編輯器使用 CodeMirror。直接對隱藏 textarea 呼叫 `.fill()` 可能不是取代全文，而是把新稿附加在舊稿後方，造成整篇重複。
- 在 macOS／Chrome 的這個操作環境，`Control+A` 沒有選到 CodeMirror 全文，`Meta+A` 才有作用。
- 大型原圖直接上傳可能長時間卡住並留下「無法顯示圖片」placeholder。將長邊縮到約 2000 px 後再上傳較穩定。

## 整篇取代流程

1. 先保留一份完整、乾淨的本地 Markdown 作為 canonical draft。
2. 聚焦 CodeMirror 編輯區，送出 `Meta+A`。
3. 在輸入新稿前，檢查 `.CodeMirror-selected` 確實存在；沒有選取就停止，不要繼續輸入。
4. 一次輸入完整乾淨稿，避免用 `.fill()` 猜測編輯器行為。
5. 等待 HackMD 儲存完成，再重新開啟公開頁面驗證。

## 完成條件

- 主標題只出現一次。
- 各主要段落標題只出現一次，結尾區塊只出現一次。
- 公開頁面沒有「無法顯示圖片」。
- 每張圖片都要確認實際載入，而不只檢查 Markdown 裡存在 URL。
- 如果補圖來源很大，先縮圖再上傳；不要把來源相簿連結、EXIF、GPS 或暫時憑證寫入公開內容或 repository。

## 憑證處理

- 若曾為一次性更新建立 API token，使用完立即撤銷。
- 任何 token 值、登入資料或 session identifier 都不得寫入檔案、commit message、終端輸出摘要或長期記憶。
- 某次執行環境連不到 HackMD API 只是暫時性 runtime 狀態，不應保存成長期平台結論。
