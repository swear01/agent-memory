---
title: Authenticated browser form continuity and submission gates
scope: global
status: active
created: 2026-09-03
updated: 2026-09-03
tags:
  - browser
  - forms
  - privacy
  - submission
---

# Form continuity

使用者已明確授權代填表單後，先完成所有可逆的欄位、選項與檢視步驟；不要每填一個欄位就重新詢問。使用者後續更正資料時，直接以新值取代舊值並繼續流程，不要重問已確定的欄位。

需要同意聲明或個資處理確認時，只在該安全邊界集中確認一次。圖形驗證碼、Email 驗證或 OTP 是人工交接點；讓使用者只處理必要的驗證，不要求其提供密碼或一次性驗證碼，完成後接續剩餘流程。正式送出、付款或其他不可逆操作前，再做一次最後確認。

頁面卡住時，先等待、停止載入或重新檢查目前頁面，優先保留已填資料；不要因一次控制元件失敗就反覆停住或清除整份表單。若必須改由本機瀏覽器接手，交接內容要包含網址、已完成／未完成步驟、欄位值、正文與最後應停下的不可逆操作。

# Evidence and privacy boundary

填完欄位、進入檢視頁或收到 `started` 類回應，都不等於正式送出。完成後要看到官方受理畫面、案件編號／查詢碼或可對應的確認通知，才能宣稱送出成功。

不要把姓名、電話、地址、Email、帳號、驗證碼、Cookie、登入狀態或私人表單正文寫入 shared memory；只保存可重用的流程規則與去識別化的失敗教訓。
