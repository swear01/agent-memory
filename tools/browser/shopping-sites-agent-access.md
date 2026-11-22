---
title: "購物網站對 agent 的可用性（Amazon、Shopee、露天、淘寶）"
scope: tools/browser
status: active
updated: 2026-11-22
---

# 購物網站對 agent 的可用性

2026-11 幫使用者比價、找貨時實測（Claude 桌面 app 內建瀏覽器 + Android adb）：

- **Amazon**（.com / .co.jp / .fr）：擋 AI agent，頁面拒絕載入。不要重試，改用 WebSearch 摘要或第三方比價頁取得價格。
- **Shopee 台灣**：內建瀏覽器開搜尋頁會出現流量驗證頁。不要嘗試繞過；請使用者自己在手機或瀏覽器查。
- **露天（Ruten）**：搜尋冷門品牌型號（例如 ATKey.Pro）常回傳不相關商品（iPhone 等），結果不可當作「沒有販售」的證據，要逐筆看標題。
- **淘寶**：用 adb 控制手機 App 比網頁穩。搜尋可直接開 deep link：
  `adb shell am start -a android.intent.action.VIEW -d 'taobao://s.taobao.com/search?q=<URL-encoded 關鍵字>'`。
  滑塊驗證交給使用者；手機會自動鎖屏，長流程中間要先喚醒。結帳、付款一律由使用者自己做。
- **價格時效**：DigiKey 等通路的價格頁可能有價無貨，回報前要確認庫存欄位，不要把舊價格當作可買。
