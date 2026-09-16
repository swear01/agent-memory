---
title: "遠端預覽不能把客戶端 localhost 當伺服器"
scope: tools/computer-use
status: active
updated: 2026-09-16
evidence_digest: 204c822f6b8664375cd22e7098ea2232f6ad74be9dac026469c0e2aa3b1f883b
---

# 遠端預覽不能把客戶端 localhost 當伺服器

使用者回報 127.0.0.1 拒絕連線，並說明正在遠端使用，要求可遠端存取的預覽。來源沒有伺服器啟動參數，因此無法確定是只綁 loopback、服務停止或其他原因。

先辨識瀏覽器與服務各在哪台機器，再設定適當的已授權連線方式並從實際客戶端驗證。不要直接把 loopback 連結交給另一台電腦，也不要無條件將服務公開綁定來掩蓋拓樸問題。來源沒有恢復連線證據。
