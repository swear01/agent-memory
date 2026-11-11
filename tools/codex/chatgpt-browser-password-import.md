---
title: ChatGPT 桌面瀏覽器可匯入密碼 CSV，與網頁 Passwords 分開
scope: tools/codex
status: active
updated: 2026-11-11
---

2026-11-11 在 macOS ChatGPT 桌面版實測，內建瀏覽器的密碼管理工具可批次匯入 CSV。匯入成功後，重新整理 ChatGPT 網頁 Passwords，其筆數未變。不要把兩個保管庫視為同一個，也不要把網頁版的限制套到桌面版。

- **桌面 CSV**：內建瀏覽器右上角「⋯」→「瀏覽器設定」→「自動填入和密碼」→「密碼管理工具」→「管理」→ 密碼管理工具內的「設定」→「匯入密碼」→「選取檔案」。可使用 Chromium 格式的 CSV（`name,url,username,password`）。以匯入完成畫面的筆數驗證結果；若出現衝突提示，未獲授權時不覆寫。
- **桌面設定檔匯入**：同一個「⋯」選單的「匯入 Cookie 和密碼」是另一個入口。當時介面只偵測 Chrome 或 Atlas 設定檔；找不到來源時，改走上面的 CSV 入口。不需要匯入 Cookie 或其他資料。
- **網頁 Passwords**：在 [設定頁](https://chatgpt.com/settings/passwords) 實測只有搜尋與重新整理，未見批次匯入。[官方桌面瀏覽器說明](https://learn.chatgpt.com/docs/browser#use-the-desktop-browser)將設定檔匯入列為桌面功能；[網頁登入說明](https://learn.chatgpt.com/docs/dots/computers-and-apps#saving-and-reusing-a-login)只確認支援登入流程在出現 **Save to Passwords** 時逐筆儲存。網頁功能與方案、工作區或發布進度可能有關，遇到不同介面時重新驗證。
