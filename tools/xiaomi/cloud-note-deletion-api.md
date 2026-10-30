---
title: 小米雲筆記批次刪除 API 與回收站機制
scope: tools/xiaomi
status: active
updated: 2026-10-30
tags: [xiaomi, mi-note, api, cookie, cdp]
---

# 小米雲筆記批次刪除 API 與回收站機制

## 核心端點與 Payload

小米雲筆記（`us.i.mi.com`）的筆記刪除有分「移至回收站（最近刪除）」與「永久清除」兩階端點：

- **移至回收站（推薦）**：
  - 端點：`POST https://us.i.mi.com/note/full/{id}/delete`
  - Headers：
    - `Content-Type: application/x-www-form-urlencoded; charset=UTF-8`
    - Session Header: Cookie 需包含 `<service-token>` 與 `<user-id>` 等欄位
  - Body 格式：
    ```text
    data={"entries":[{"id":"<note-id>","tag":"<note-tag>"}]}
    ```
    - 注意：URL 中的 `{id}` 為欲刪除之筆記 ID，但 POST body 內亦必須攜帶 `data` 欄位，且其內部 `entries` 陣列包含對應的 `id` 與 `tag`（版本戳記）。若漏傳 `tag` 或 body 為空，服務端會回傳格式錯誤或略過。
    - 成功回應範例：`{"result":"ok","code":0,"data":{"entry":{"id":"...","status":"deleted",...}}}`。筆記移入「最近刪除」，30 天內可手動或透過 API 復原。

- **永久清除（Purge）**：
  - 端點：`POST https://us.i.mi.com/note/full/{id}/purge`
  - 效果：不經過回收站直接物理抹除，無法挽回。除非使用者明確要求徹底銷毀，否則自動化腳本應一律呼叫 `/delete`。

## 認證與 Session 期限

- 小米雲端認證依賴 Cookie 中的 serviceToken 與 userId。
- 實測 serviceToken 有效期約數小時至半天（約 6 小時）。過期時呼叫任何筆記 API 會回傳 HTTP 401 或 JSON `{"code":99999,"description":"not logged in"}`。
- 自動化維護方式：透過已登入之瀏覽器（例如 Brave / Chrome）連線 CDP，利用 `Network.getCookies` 針對 `https://us.i.mi.com` 提取最新的 serviceToken，寫入本機腳本的環境設定（如 `<project-root>/env`），避免在腳本中明文儲存帳號密碼。
