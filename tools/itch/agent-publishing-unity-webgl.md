---
title: 從 agent session 發布 Unity WebGL 到 itch.io（butler、圖片上傳、主題截圖）
scope: tools/itch
tool: itch.io
status: active
created: 2026-10-20
updated: 2026-10-20
tags: [itch.io, butler, Unity, WebGL, unityweb, Content-Encoding, browser-upload, jam]
---

2026-10-20 發布 Gamejam1 PHASE 時實測的經驗；之後的 itch 介面或工具版本可能改變，仍需重新驗證。

## 上傳大型 HTML5 zip

- Claude in Chrome 的 `file_upload` 單次總大小必須小於 10 MB，18 MB 的 Unity WebGL zip 無法用它上傳。
- 內建瀏覽器（Claude desktop browser pane）沒有檔案上傳工具；從 `https://itch.io` 頁面 `fetch('http://127.0.0.1:…')` 會 `Failed to fetch`，iframe 也被擋；`window.open` 會在同一個 tab 導航，未存檔的表單內容會遺失。
- 可行做法是 itch 官方 butler：`https://broth.itch.zone/butler/darwin-arm64/LATEST` 查版本，下載 `/<version>/archive/default`，解壓到 `~/.local/share/butler/<version>/`（需保留 `7z.so`、`libc7zip.dylib`），再把 `~/.local/bin/butler` 連到它。15.32.0 由 `Developer ID Application: itch corp. (AK2D34UDP2)` 簽章並經 notarize。Homebrew 的 `butler` cask 是 Many Tricks 的無關 App。
- `butler login` 會開瀏覽器授權，由使用者本人執行；之後 `butler push <zip> <user>/<game>:html5 --userversion <commit>`。新 channel 的 upload 會自動勾「This file will be played in the browser」。`butler status` 的 `<` 代表仍在處理，API 也可能 timeout；以遊戲頁 iframe 出現 `html-classic.itch.zone/html/<upload>-<build>/` 判斷完成。

## itch CDN 對 Unity 壓縮檔的處理

- itch 依內容偵測 gzip，`.unityweb`（gzip）會帶 `Content-Encoding: gzip`，`.wasm.unityweb` 的 `content-type` 是 `application/octet-stream`；`index.html`、`.js` 也會被 CDN gzip。
- Unity gzip + Decompression Fallback 在 itch（瀏覽器原生解壓）與無 header 的靜態伺服器（JS fallback）都能正常載入，不需要改 Player 設定。
- Unity data caching 重新驗證時，`Web.data.unityweb` 會出現 304 + `net::ERR_ABORTED`，每次都伴隨 `[UnityCache] ... successfully revalidated and served from the browser cache`，屬正常。Playwright `request.headers()` 看不到 `if-none-match`，若驗證腳本靠它判斷 conditional request 會誤判。

## 封面、截圖與頁面設定

- itch 的封面與截圖按鈕在點擊時才建立 `<input type=file>`，沒有可供 `file_upload` 指定的常駐 input。使用者要求 agent 自己上傳時，可在已登入的內建瀏覽器中暫時覆寫 `HTMLInputElement.prototype.click`：對 file input 用 `DataTransfer` 填入 `File` 並 dispatch `change`，用完立即還原（拋錯也要還原，否則重新整理頁面）。圖片資料以 base64 傳入；像素圖可傳原生解析度，再用 `imageSmoothingEnabled=false` 的 canvas 放大後 `toBlob`。
- 手抄 base64 不可靠：把工具輸出的 base64 重打進 `javascript_tool`，4 KB 左右就可能抄錯。損壞的 PNG 讓 itch 縮圖回 500；瀏覽器部分解碼後再用 canvas 重新編碼，雖然能上傳，但下半部是空白（PHASE 首次封面 630×500 只有上方約 180 列有內容）。做法：分成約 400 字元的區塊，每塊附 SHA-256 前綴，在頁面內驗證，不合格的區塊重送；組合後再比對整個檔案的 SHA-256，以及 RGBA 像素的 SHA-256。
- 像素圖先縮成精確的原生解析度再傳，可大幅減少要抄的內容（例：630×500 封面是 315×250 的精確 2 倍，再用 105 色精確調色盤只剩 3.3 KB）。在頁面內放大時用 ImageData 迴圈逐像素複製；Chrome `drawImage` 搭配 `imageSmoothingEnabled=false` 做 10 倍放大並不精確。用 PIL 檢查「是否為精確放大」時要比對 `tobytes()`，因為 RGBA 影像的 `ImageChops.difference(...).getbbox()` 只看 alpha，會漏掉 RGB 差異。
- 上傳後要確認 `game[cover_image_id]` 有值、縮圖 `naturalWidth > 0`，存檔後再下載 og:image 原圖，比對像素與檢查空白列。
- 嵌入寬 960 時，主題 Layout → Screenshots 的 `Auto` 會解析成隱藏（`.view_game_page .right_col{display:none}`）。在 Edit theme 選 `Sidebar` 並存檔，截圖才會出現在說明旁。
- Embed 寬高欄位由 widget 控制，直接設定 value 不生效；需真實點選並輸入。描述用 Redactor，可用 `$(textarea).redactor('code.set', html)`，再讀回具名 `game[description]` 確認。
- Jam「Submit your project」對話框的專案 `<select>` 用一般 `form_input` 不會更新 React 狀態；用原生 `HTMLSelectElement.prototype.value` setter 設值並 dispatch `change`，確認 Requirements 顯示通過後再按 Submit。公開與投稿都要使用者逐項明確同意。
