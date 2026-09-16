---
title: "首頁 200 不能代替指定站台路徑验收"
scope: tools/web
status: active
updated: 2026-09-16
evidence_digest: 57bbd4d1c6c0147f54e9163f6ea9ba78a26ab57fdc8618888abd96e31341dbf9
---

# 首頁 200 不能代替指定站台路徑验收

來源顯示 nginx active 且根路徑200，但 dashboard、overview 與 JSON 路徑全404；歷史助手承認先誤判成功、關掉預覽後才發現站台設定可能未生效。

核對實際目標路徑、內容與 listener／站台配置，再移除本次臨時預覽。服務活著與預設頁可開只能證明各自範圍；來源沒有後续站台恢復結果，也不把其他主機離線混成同一部署問題。
