---
title: Internet Archive S3 上傳（ia CLI，網頁 uploader 卡住時的對策）
scope: tools
status: active
created: 2026-09-21
updated: 2026-09-21
tags: [internet-archive, archive-org, ia-cli, s3, upload, mac]
---

# Internet Archive 上傳（Mac）

## 網頁 uploader 在 Mac 上常卡

`archive.org/upload/` 網頁的 S3 uploader 在 macOS（尤其 Brave + CDP 操作）常卡住：送完檔後網路請求歸零、出現 `Please enter a valid web address` / `Resume Uploading` 對話框，檔案傳完但 item 建不出來。而且重載頁面會清空已填的 metadata。

**改走 `ia` CLI 走 S3，一次成功**（本輪 2026-09-21 驗證）：

```
uv tool install internetarchive
```

## 抓 S3 keys（不需密碼）

登入的 IA 網頁 uploader 有 `.js-uploader-args` hidden input，`JSON.parse(value).s3user` 含該帳號的 `s3accesskey`/`s3secretkey`（帳號常規 S3 keys，非 session-scoped）。用 CDP `Runtime.evaluate` 讀出。

## 用 ia CLI 上傳（關鍵：先複製到本地）

`ia` CLI 讀 **Google Drive 同步（DriveFS）的檔案**會觸發 `OSError: [Errno 11] Resource deadlock avoided`（見 drivefs 筆記）。**先 `cp`/`Preview` 水合到本地再傳**：

```
ia upload <identifier> /本地/檔案 \
  --metadata "title=..." --metadata "creator=..." \
  --metadata "description=..." --metadata "subject=..." \
  --metadata "language=jpn" --metadata "collection=opensource" \
  -R 5 -s 5
```

- 參數順序：`identifier` 在前、檔案路徑在後（放錯會 `argument identifier: ... not a valid file`）。
- 用 **ASCII identifier**（如 `dai-gaku-nihongo-tomodachi-vol2e`），別用含 `.` 的。
- 成功進度條會跑到 `100%| 76/76`；CLI 回 `success: <identifier> is accepting requests`。
- 之後 `ia metadata <identifier>` 或 `curl https://archive.org/metadata/<identifier>` 驗證 files + title/creator/language。
- 下載驗證：`curl -sL -o out.pdf https://archive.org/download/<identifier>/<filename>`（注意實際檔名是上傳時的檔名，不是 identifier）。
- 純 `--s3` PUT 要 bucket 先存在，會 404；`ia upload` 會自動建 item + bucket 再 PUT，所以直接用它最省事。
- 上傳中斷重跑：先 `ia upload` 建 item（會建 bucket），之後同 identifier 重傳即可續。

## ia CLI 的 config

`ia configure -C`（validate S3 keys）/ `-w`（whoami）需要 keys。設環境變數 `IA_ACCESS_KEY_ID` + `IA_SECRET_ACCESS_KEY` 就能用，不用寫 config 檔（config 檔要 `[general] screenname=` 否則 `parse_config_file` 報 `No option 'screenname'`）。

## Playwright 連 CDP 的 50MB 限制

`playwright connectOverCDP` + `filechooser.setFiles` 對 >50MB 檔案報 `Cannot transfer files larger than 50Mb to a browser not co-located with the server`。大檔要改用 raw CDP `DOM.setFileInputFiles`（無 50MB 限制）：先 `DOM.enable`、`DOM.getDocument`、`DOM.querySelector('#file_input_initial')` 拿 nodeId，再 `setFileInputFiles`。但 IA 網頁 uploader 本身不稳，最終仍建議走 ia CLI。

## 實例

- 2026-09-21：《大学の日本語 ともだち Vol.2》(76MB PDF, Honto 除 DRM) 用 `ia` CLI 傳到 `dai-gaku-nihongo-tomodachi-vol2e`，`book.pdf` 79591285 bytes / 370 頁上線。