---
title: "活動日期缺值不能默默改成 render 當天"
scope: tools/media
status: active
updated: 2026-09-16
evidence_digest: e2650648e331d9d215bb14505534ea7b70d298b31bf9d1b222fdf81addc0ce96
---

# 活動日期缺值不能默默改成 render 當天

歷史唯讀調查發現多個 reveal config 的 date 為空，影片與公告模板 fallback 到 new Date；成品因此顯示 render 日，而部分 update image config 又有不同日期。

將必填活動日期與 render 時間分開，驗證影片、圖片與正式排程一致，再讀回成品。使用者後續說每個門間隔三天，只提供間距，仍須有已確認起點；官方實裝日也不是活動日。來源只到診斷及授權修正，沒有最後補值或重製通過證據。
