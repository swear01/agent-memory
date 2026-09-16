---
title: "Pi 顯示名稱與 HAPI title event 是不同資料"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 2c39736847607d3a863202cf95e1d88473b3217156a2c14cfbb91d4fc36e7055
---

# Pi 顯示名稱與 HAPI title event 是不同資料

歷史助手先說 Pi 不會自動改 HAPI 標題，後來收窄為 Pi 可有 session name 或第一則訊息的顯示名稱，但當時 adapter 不一定把它同步到 HAPI metadata；討論同時区分 terminal title、session name 與模型摘要。

沿實際 RPC producer、adapter 消費事件與 HAPI 顯示欄位確認同步。設定環境旗標或有接收程式碼，不代表 producer 已發事件；也不能把沒有摘要功能說成完全沒有名稱。來源只是指定安裝版的閱讀回報，沒有修改或端到端驗證，不推論目前 Pi 能力。
