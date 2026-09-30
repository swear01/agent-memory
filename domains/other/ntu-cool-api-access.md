---
title: "NTU COOL（Canvas）API 存取與隱藏權杖 UI"
scope: "domains/other"
status: active
confidence: high
updated: 2026-09-29
tags:
  - ntu-cool
  - canvas
  - api
  - access-token
---

# NTU COOL（Canvas）API 存取與隱藏權杖 UI

## 帳號

研究所與大學部 COOL 帳號由同一人持有，課程與權杖彼此獨立；數位系統設計使用大學部帳號。

## API

- Base：`https://cool.ntu.edu.tw/api/v1`
- 認證：`Authorization: Bearer <personal access token>`
- 不需 VPN 即可打 COOL API；ADFS 與其他校內站點才需要本機 VPN 設定。

## 權杖存放

兩帳號的權杖分別存於本機 mode 600 secrets 檔；公開記憶不記錄帳號、具體路徑或權杖。權杖用途名稱可在 COOL 設定頁辨識與撤銷。

## 產生權杖：UI 被 CSS 隱藏，不是沒有

NTU 在 `https://cool.ntu.edu.tw/profile/settings` 把「已核准的整合」藏起來：

1. DOM 仍有標題「已核准的整合」與 `a.btn.btn-primary.add_access_token_link`（文字「新訪問令牌」）。
2. 父層 `.approved_integration_content` 設了 `display: none`，所以畫面上看不到。
3. 用頁面腳本把 `.approved_integration_content` 改成 `display: block`。
4. 點「新訪問令牌」→ 填用途 → 到期留空 → 產生。
5. 權杖只寫入 secrets 檔（mode 600），絕不截圖、不寫進對話或 memory 正文。

## 台大單一登入（ADFS）表單填寫

- 入口：`https://cool.ntu.edu.tw/login/saml`
- 欄位用 CSS selector（snapshot ref 會過期）：
  - `#ContentPlaceHolder1_UsernameTextBox`
  - `#ContentPlaceHolder1_PasswordTextBox`
  - 送出：`#ContentPlaceHolder1_SubmitButton`
- 欄位在頂層頁面，不在公告 iframe 內。

## 讀寫原則

讀課程／檔案／公告／作業優先用 API。交作業、討論區發文、寄信、改設定須使用者明確確認。
