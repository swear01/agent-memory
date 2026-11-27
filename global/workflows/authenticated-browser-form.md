---
title: Authenticated browser form continuity and submission gates
scope: global
status: active
created: 2026-09-03
updated: 2026-11-28
tags:
  - browser
  - forms
  - privacy
  - submission
---

## 帳號與表單

- **自動化視窗顯示登入，使用者日常環境卻可用。** 優先官方 API／已連線 connector，需要互動時用 Codex 內建瀏覽器。先核對控制面、既有登入或保管庫；私人 Brave 或其他外部 browser 須依使用者指定或授權。別搬 cookies、清 profile 或展示憑證。
- **表單填好了，還沒正式受理。** 先完成可逆欄位。更正用最新值；處理必要聲明和人工 CAPTCHA／OTP 交接後續做，不請使用者貼秘密。已授權送出且目標未實質改變時直接送出。用官方確認頁、案件編號或同一次的收件通知驗證。失靈先觀察現有狀態，保留已填資料，避免重複提交。
- **控制面逾時但頁面可能仍在執行。** `Emulation.setFocusEmulationEnabled` timeout 與 `noWindowsAvailable` 曾只影響一種控制面；核對 URL、可見結果和載入，必要時換已授權的既有控制面。Sparkle 更新副本會讓 bundle ID 變模糊；可用主 App 路徑。PDF 預覽按下載後還要驗本機檔案和內容。
- **Drive 搜尋只出現回覆表。** 用內建瀏覽器的 Google Forms 首頁找原始表單，確認檔案類型，用工具取正確 ID。編輯後等雲端保存，再重新載入驗題名、選項及「其他」；歷史回覆仍可有舊字串。要查今天時分清提交時間和消費日期，與連結試算表核對，勿增加假消費。

## 附件

- **Photos 剛上傳找不到中文關鍵字。** 用影像上的英文或「最近新增」定位，單張 `Shift+D` 下載，再核對新檔格式與內容。
- **Forms 檔案 picker 讀得到、卻不能用頁面 JS 塞本機檔案。** 頁面對 loopback 的 fetch 曾回 `Failed to fetch`。已驗證可放進既有 Google Drive for desktop 同步目錄，再在 picker 的「我的雲端硬碟」選取。內建瀏覽器 `window.open` 曾取代原分頁，草稿恢復後重查單選與附件。送出後確認已複製到表單，也再決定清理備用檔。只收 PDF 時，先合併、逐頁驗證。
- 共用記憶只留去識別規則。私人表單網址、姓名、聯絡資料、代碼與正文留在受控來源。

[整理前完整證據](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/global/workflows/authenticated-browser-form.md)
