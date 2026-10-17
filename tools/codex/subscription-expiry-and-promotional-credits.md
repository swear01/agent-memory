---
title: ChatGPT 訂閱精確期限與 62500 過渡贈點的查核方法
scope: tools/codex
status: active
updated: 2026-10-17
---

## 精確訂閱期限

- 郵件與 Stripe 帳務頁可能只顯示日期，甚至顯示不同日期；不能推定到期日午夜失效。
- 2026-10-17 實測：已登入的 ChatGPT 網頁自然載入 `backend-api/subscriptions` 回應，可讀取 `active_until`、`entitlement.renews_at` 與 `scheduled_plan_change.plan_type`。兩個期限欄位的 UTC 時刻一致，可換算成所需時區。
- DevTools Network 篩選 `subscriptions`，查看 Response／Preview；這是非公開介面的觀察方法，欄位與端點可能變動，每次以實際回應為準。
- 只查訂閱相關欄位，不搬移登入憑證，不保存付款 session 連結、帳號識別資料或完整網路紀錄。個人訂閱快照留在私有本機記憶與 QMD。
- 後台排定時間不等於已驗證實際切換；先前郵件日期應被較精確的當次帳號回應取代。

## 公開的 62500 credits 過渡贈點資訊

- 收件者公開通知稱：舊 Pro $200 的 Work／Codex 額度於 2026-10-30 從 Plus 的 20 倍改為 10 倍，附一次性 62,500 點過渡贈點，至 2026-12-31 到期。
- 官方 Pro 文件確認符合資格的舊 Pro 200 可保留舊額度至 10/29，之後額度調低；公開資料未完整說明每個帳號的贈點資格，不能保證所有 Pro 用戶均獲贈。
- 官方學生頁明載 2,500 點相當於 US$100；所以 62,500 點名目使用價值為 US$2,500。學生方案只用於確認換算，不能據此判斷該過渡贈點的資格或期限。
- GPT-6.1 Sol Standard 一般輸入／輸出的點數費率與 API 美元費率可作量級比較；API 的快取寫入、長上下文、工具與速度模式會造成差異。這些點數不是 API 餘額，也不能兌現。
- 官方通用規則：先消耗方案內含額度，再用 credits；模型與功能可用性仍受目前方案限制。
- 同批用戶公開轉述客服書面答覆：降級不撤回未用贈點，期限仍為 12/31。這是客服答覆的社群轉述，不能說成對每個帳號的實測保證；降級後須讀回餘額與可用狀態。

## 來源

- 精確時間實作：https://bbs.kanxue.com/thread-291986.htm
- 官方 Pro 說明：https://help.openai.com/en/articles/9793128-about-chatgpt-pro-tiers
- 收件者通知：https://www.reddit.com/r/OpenAI/comments/1wtlpx7/its_been_a_good_run/
- 降級客服答覆轉述：https://www.reddit.com/r/ChatGPTPro/comments/1wvdhuu/has_anyone_kept_the_62500_promo_credits_after/
- 官方點數規則：https://help.openai.com/en/articles/12642688-using-credits-for-flexible-usage-in-chatgpt-personal-plans
- 官方換算：https://learn.chatgpt.com/community/students
- Codex 點數費率：https://learn.chatgpt.com/docs/pricing
- API 費率：https://developers.openai.com/api/docs/models/gpt-6.1-sol
