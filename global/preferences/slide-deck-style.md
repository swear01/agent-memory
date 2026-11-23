---
title: 簡報一律黑底
scope: global slide decks
status: active
updated: 2026-11-24
tags:
  - slides
  - presentation
  - style
---

使用者明確要求：幫他做的所有簡報（Claude Slides artifact、HTML、pptx 皆同）一律統一黑底，不要混用淺色底或整頁亮色強調頁。

- 每一頁都用深色背景，包含封面、強調句頁、結語頁。需要強調時改用強調色文字（例如橘色大標），不要換成亮色整頁背景。
- 2026-11 用過且使用者接受的配色：底 `#15171D`、卡片 `#1F222A`、分隔線 `#3A3E4A`、主文字 `#F3F0E8`、次要文字 `#B9BCC6`、強調色 `#FF8A4C`。字型 Noto Sans TC（標題 900）＋ Space Grotesk（數字）。
- 從淺色改黑底時，除了 deck.css 的變數，還要搜尋 slide 檔裡寫死的 inline 深色文字（如 `color:#3A2A20`），否則黑底上會看不見。
- 不要整份只有字：提到具體遊戲的頁面要配該遊戲的官方圖。Steam 遊戲用 `store.steampowered.com/api/storesearch` 找 appid，再用 `api/appdetails` 的 `header_image`（460×215，卡片寬約 450px 時剛好）和 `screenshots`；新上架的遊戲 `capsule_616x353.jpg` 常常 404。非 Steam 遊戲找 Wikimedia Commons 的自由授權圖，並在資料來源頁標授權。抽象論點頁可以維持純文字。
- 匯出單一 HTML 時，artifact 的 `/_blob/<id>` 圖片要換成 base64 內嵌；全版背景圖在瀏覽器裡要加 `z-index:-1`（section 設 `isolation:isolate`），不然會蓋住標題。
- 遊戲設計研究社的簡報交付時，除了 artifact，另匯出單一 HTML 放到 Google Drive `document/工作紀錄/遊戲設計研究社/<簡報名>/<簡報名>.html`，與既有的 TIS-100、前程似錦簡報同格式。

相關：`domains/other/game-analysis-talk-deck-workflow.md`
