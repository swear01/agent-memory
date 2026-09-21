---
title: Z-Library FTP Collection 上傳流程與 0 字節錯誤根因
scope: tools
status: active
created: 2026-09-21
updated: 2026-09-21
tags: [z-library, ftp, upload, ebook, publisher, non-ascii-filename]
---

# Z-Library FTP Collection 上傳（大檔案 / 網頁上傳卡住時用）

Z-Library 網頁的 drag-drop 上傳在 76MB 這種檔案常卡在 `Uploading...` 不動（瀏覽器端 PUT 卡住）。>100MB 才強制用 FTP，但大檔案走 FTP 更穩。

## 關鍵事實（都踩過坑驗證）

- **建立 FTP collection**：`/uploader/collections` → Create FTP collection → 名稱 + content type（book）。POST `/papi/uploader/collection/create`（`name` + `content_type=book`）回 `{id, dir:"collectionXXXX/", ftp_user_id}`。
- **FTP 憑證**：collection 列的 `Credentials` 連結彈出 `ftp-access-popup`。host `95.217.88.17`、port 21、username = `collectionXXXX`、password 為隨機字串。
- **檔案要放 FTP 根目錄**（`/`），不是 `collectionXXXX/` 子目錄。Z-Library 掃的是根目錄。
- 用 curl FTPS 上傳：`curl --ftp-ssl "ftp://collectionXXXX:pass@95.217.88.17:21/" -T /本地/檔案`（成功是 `226 Transfer complete` + `upload completely sent off: N bytes`）。
- 上傳完到 collection-view 點 `Start processing` → `PROCESSING` → `READY`。

## 0 字節 / Unknown error 根因（最重要）

**非 ASCII 檔名（中日文檔名）會讓 Z-Library 讀到 `0 B`、狀態 `Error`（`Unknown error`）。** 處理程式用該檔名開不到檔案。修法：
1. 檔案改成**純 ASCII 檔名**（如 `book.pdf`）再上傳。
2. 若舊 collection 已追蹤那個非 ASCII 檔名的 file-id（顯示 0B/Error），retry 也修不好——**重建 collection**（DELETE 舊的、CREATE 新的拿新 `collectionXXXX` 帳號），用 ASCII 檔名重傳，一次就好。
- 注意 curl 對非 ASCII 遠端檔名的 DELE/上傳會因編碼不一致而靜默失敗；改 ASCII 檔名可繞過。

## Ready to publish → Published 前要填 metadata

檔案處理好後狀態常是 `Fill out info`（`You have to fill mandatory book info`），不是直接 ready。要補必填欄：
- 點該列的 `.btnBookEdit` 開 `form.edit-book-container` modal。
- 必填：**Title、Author、Language**（Language 是 hidden `select.multiselect__target-select`，JS 設 `value='japanese'` + dispatch change）。
- 建議填：Publisher、Year、Pages。Category 是 multi-select、**非必填**可留空。
- 用 `Object.getOwnPropertyDescriptor(proto,'value').set.call(el, val)` + `input`/`change` 事件填值（React/JS widget 不吃 `el.value=`）。
- 點 modal 的 Save → 變 `Ready to publish` → 勾該列 checkbox → `Publish whole collection` → `Published`。
- **BASIC 帳號可以 publish**（本輪 2026-09-21 成功上架，未升級）。
- 站內提示：未發布檔案 30 天自動刪除，所以處理完要盡快 publish。

## 操作 CDP 的坑（這台 Mac）

- 走 raw CDP（`Target.attachToTarget` + flatten session）對 z-library 頁面常卡 `Runtime.evaluate` timeout / `Target crashed`；瀏覽器-level WS（`/json/version` 的 `webSocketDebuggerUrl`）較穩。
- 用**同 session id**：attach 回傳的 `sessionId` 要帶在後續 `Runtime.evaluate` 的 `sessionId` 參數。attach 兩次會拿到不同 sid 而 `Session with given id not found`。
- 卡住就 `Target.closeTarget` 關掉重開新 target；`Network.getAllCookies` 要 session-scoped，browser-level 會 `wasn't found`。

## 實例

- 2026-09-21：《大学の日本語 ともだち Vol.2》(76MB PDF, Honto 除 DRM) 用 FTP collection `collection27425`（ASCII `book.pdf`）+ 填 metadata → Z-Library Published。