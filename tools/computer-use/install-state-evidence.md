---
title: "權限已儲存不等於應用程式已安裝"
scope: tools/computer-use
status: active
updated: 2026-09-16
evidence_digest: d229bca1dbf24ac3e66800206f630f49cc35db19dc4a153d10111c7c54e018f1
---

# 權限已儲存不等於應用程式已安裝

歷史助手先稱缺少 webhooks 權限，後發現 UI 沒有該項控制；接著安裝頁因 repo selection 未生效而讓 Install 按鈕停用。

依當前 UI 與實際授權模型分別確認權限、目標選取、安裝完成及需要的事件交付。不要虛構不存在的控制項，也不能用部分設定成功代表全部安裝完成。來源沒有安裝收據或 webhook 交付結果。
