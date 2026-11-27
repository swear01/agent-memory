---
title: 從 agent session 發布 Unity WebGL 到 itch.io（butler、圖片上傳、主題截圖）
scope: tools/itch
tool: itch.io
status: active
created: 2026-10-20
updated: 2026-11-25
tags: [itch.io, butler, Unity, WebGL, WebGPU, unityweb, Content-Encoding, browser-upload, theme, jam]
---

2026-10-20 發布 Gamejam1 PHASE 時實測的經驗；之後的 itch 介面或工具版本可能改變，仍需重新驗證。

## 上傳大型 HTML5 zip

- Claude in Chrome 的 `file_upload` 單次總大小必須小於 10 MB，18 MB 的 Unity WebGL zip 無法用它上傳。
- 內建瀏覽器（Claude desktop browser pane）沒有檔案上傳工具；從 `https://itch.io` 頁面 `fetch('http://127.0.0.1:…')` 會 `Failed to fetch`，iframe 也被擋；`window.open` 會在同一個 tab 導航，未存檔的表單內容會遺失。
- 可行做法是 itch 官方 butler：`https://broth.itch.zone/butler/darwin-arm64/LATEST` 查版本，下載 `/<version>/archive/default`，解壓到 `~/.local/share/butler/<version>/`（需保留 `7z.so`、`libc7zip.dylib`），再把 `~/.local/bin/butler` 連到它。15.32.0 由 `Developer ID Application: itch corp. (AK2D34UDP2)` 簽章並經 notarize。Homebrew 的 `butler` cask 是 Many Tricks 的無關 App。
- `butler login` 會開瀏覽器授權，由使用者本人執行；之後 `butler push <zip> <user>/<game>:html5 --userversion <commit>`。新 channel 的 upload 會自動勾「This file will be played in the browser」。`butler status` 的 `<` 代表仍在處理，API 也可能 timeout；以遊戲頁 iframe 出現 `html-classic.itch.zone/html/<upload>-<build>/` 判斷完成。2026-11-25 實測：四個 build 推完兩小時後 `butler status` 仍顯示 `<`，但公開頁已能下載、iframe 也能載入，所以別一直等 `<` 消失。

## itch CDN 對 Unity 壓縮檔的處理

- itch 依內容偵測 gzip，`.unityweb`（gzip）會帶 `Content-Encoding: gzip`，`.wasm.unityweb` 的 `content-type` 是 `application/octet-stream`；`index.html`、`.js` 也會被 CDN gzip。
- Unity gzip + Decompression Fallback 在 itch（瀏覽器原生解壓）與無 header 的靜態伺服器（JS fallback）都能正常載入，不需要改 Player 設定。
- Unity data caching 重新驗證時，`Web.data.unityweb` 會出現 304 + `net::ERR_ABORTED`，每次都伴隨 `[UnityCache] ... successfully revalidated and served from the browser cache`，屬正常。Playwright `request.headers()` 看不到 `if-none-match`，若驗證腳本靠它判斷 conditional request 會誤判。

## 封面、截圖與頁面設定

- **首選做法（2026-11-25 驗證）：不經瀏覽器傳圖檔，改用 itch 自己的上傳流程 + shell curl**（跟 NTU COOL 同一招）。在已登入的編輯頁用頁面 JS：`POST /dashboard/upload-image?type=cover`，body（urlencoded）`csrf_token`（取 `input[name=csrf_token]` 的值；`meta[name=csrf_token]` 會回 `mismatched token`）、`filename`、`thumb_size=cover_big`、`action=prepare`、`width`、`height`。回傳 `action`（`https://itchio.commondatastorage.googleapis.com`）、`post_params`（`acl`、`key`、`GoogleAccessId`、`policy`、`signature`，一次性、約 2 小時到期，不寫進 memory）、`success_url`、`upload_id`。shell 端 `curl -X POST <action>`，每個 post_param 用 `--form-string`，最後 `-F "file=@<圖>;type=image/jpeg"`，回 204 即成功。再由頁面 JS 帶 `csrf_token` POST `success_url`，回 `{success:true, upload:{id}}`；把 `input[name="game[cover_image_id]"]` 設成該 id 後按 Save。圖檔大小不受 token 限制（上限 3 MB），可直接傳 630×500 原圖。驗證：公開 `data.json` 的 `cover_image` 換成新 id，並下載 `https://itchio.commondatastorage.googleapis.com/img/<id>.jpg` 與來源檔比對 SHA-256（本次完全相同）。截圖應可用同一流程（`type` 不同），尚未驗證。
- 封面或截圖上傳成功、`game[cover_image_id]` 有值與縮圖正常，都不能單獨證明原圖完好。從公開頁 `og:image` 取 `/original/`，查尺寸與全白／透明列，和來源比對 RGBA bytes。`ImageChops.difference(...).getbbox()` 在 RGBA 只看 alpha 可能漏掉 RGB。以像素精確放大時，逐像素複製後驗完整解碼 bytes；`drawImage` 關平滑在當時版本也可能變更像素。
- 舊版瀏覽器曾遺失手動轉碼的圖片資料，縮圖仍能載入但原檔有整片空白。歷史故障、舊處理與實際驗收見 `git show 2505b5e:tools/itch/agent-publishing-unity-webgl.md`。
- 嵌入寬 960 時，主題 Layout → Screenshots 的 `Auto` 會解析成隱藏（`.view_game_page .right_col{display:none}`）。在 Edit theme 選 `Sidebar` 並存檔，截圖才會出現在說明旁。
- Embed 寬高欄位由 widget 控制，直接設定 value 不生效；需真實點選並輸入。描述用 Redactor，可用 `$(textarea).redactor('code.set', html)`，再讀回具名 `game[description]` 確認。
- Jam「Submit your project」對話框的專案 `<select>` 用一般 `form_input` 不會更新 React 狀態；用原生 `HTMLSelectElement.prototype.value` setter 設值並 dispatch `change`，確認 Requirements 顯示通過後再按 Submit。公開與投稿都要使用者逐項明確同意。

## 多個 upload 與 embed（2026-11-25 驗證）

- 同一專案有 mac／windows／html5 等多個 upload 時，「This file will be played in the browser」可能勾到錯的 upload（本次勾到 mac zip）。結果：公開頁顯示 "The developer has not uploaded a game yet..."，mac zip 也從下載清單消失。存檔後要逐一讀 `upload[<id>][embed]`，確認只有 html5 那個是 true。
- embed 的 upload 不會渲染平台勾選框。取消 embed 後要先存檔、重新整理，`upload[<id>][p_osx]` 等勾選框才會出現，再勾平台並存檔。
- Unity 預設 WebGL 模板在 canvas 下方有約 38px 的 footer（logo、標題、全螢幕鈕）；itch embed 高度設成 canvas 高度 +40（例如 960×600 的 canvas 設 960×640），footer 才不會被切掉。
- 主題：banner 圖用 `POST /dashboard/upload-image?game_id=<id>`（流程同封面），主題欄位 `layout[bg_color]`、`layout[banner_image][image_id]` 等，整份表單 urlencoded 加 `X-Requested-With` POST 到 `https://<user>.itch.io/<game>/edit`，回 `{"success":true}`。
- 編輯頁的 `game[type]` 是 selectize：`document.querySelector('[name="game[type]"]').selectize.setValue('html')`。

## Unity WebGPU 網頁版

- shader 用 StructuredBuffer 時 WebGL2 不行，Graphics API 只放 `WebGPU`（Unity 6.x）。目前只有桌面 Chrome／Edge、Safari 26+ 能跑。
- 網頁版沒有系統中文字型，IMGUI 的中文會消失。要把 OFL 字型子集（`pyftsubset --text-file=`）放進 `Resources` 並設 `GUI.skin.font`；macOS 內建的 STHeiti 不能散布。
- 效能：在 index.html 的 `createUnityInstance` 前插入 `config.devicePixelRatio = 1;`，Retina 螢幕的像素量少四倍。
- 量效能別用隱藏的 browser pane（會降到 1 fps）。用 headless Playwright 加 Chromium（`--enable-unsafe-webgpu --ignore-gpu-blocklist`），看 console 的 `Unity WebGPU:` 與遊戲自己印的 frame time。
