---
title: BitoPro 行銷郵件退訂與設定驗證
scope: tools/bitopro
status: active
updated: 2026-09-15
---

# BitoPro 行銷郵件退訂

- 2026-09-15 實測：EDM 的「取消訂閱」追蹤連結導向官方網站 `/advanced`；未登入時再導向登入頁。這是帳戶「進階設定」入口，不能只因跳到登入頁就判定退訂連結失效。
- 登入後進入 `/advanced`，關閉「最新活動 / 行銷信通知」。對應 checkbox 為 `marketing_notice_status`，與價格通知、交易手續費選項分開；只修改使用者授權的行銷通知。
- 自訂 switch 的 `setChecked(false)` 曾逾時；確認 checkbox 仍勾選後，點擊包含該 checkbox 的 `label` 可成功切換。操作前須依當前 DOM 確認關聯，避免重複點擊而重新開啟。
- 完成門檻：重新載入頁面，讀回 `marketing_notice_status.checked === false`。這能證明設定已保存，不能證明已排程郵件立即停止。
- 本次原始郵件沒有 `List-Unsubscribe` 標頭；依 Listmonk 通用格式組成的訂閱入口回傳 404。不要將產品通用路由推定為該部署可用，優先使用已驗證的帳戶設定入口。
- 不保存收件人、subscriber/campaign UUID、追蹤連結、登入資料或郵件原文。網站設定與 DOM 可能變更，下次操作仍應重新確認。
