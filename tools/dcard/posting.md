---
title: 代發 Dcard 文章
scope: tools/dcard
status: active
updated: 2026-10-29
tags:
  - dcard
  - browser
  - sign-in
---

# 代發 Dcard 文章

- 進 https://www.dcard.tw/f/ntu 會先遇到 Cloudflare 勾選框，勾一下就過。
- 登入頁是 id.dcard.tw/oauth/login（Email＋密碼、手機、Facebook、Apple）。
- 2026-10-29：保險庫裡舊的 Dcard 密碼被判 "Incorrect password!"，使用者填的新值也一樣失敗；最後是使用者自己按「忘記密碼」重設後在 bot 的瀏覽器登入。登入失敗兩次就直接交畫面給使用者，不要一直重試。
- 發文頁：https://www.dcard.tw/new-post?type=classic&forumAlias=ntu。身份（卡稱）預設就是「國立臺灣大學 電機工程學系」；標題分類與話題都是選填。
- 流程：先把內文寫到檔案，填好停在發文前截圖給使用者確認，使用者說可以才按 Post。
