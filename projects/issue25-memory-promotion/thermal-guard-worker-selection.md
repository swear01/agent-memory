---
title: "Issue25 熱保護相依成功仍要核對 worker 選取範圍"
scope: projects/issue25-memory-promotion
status: active
updated: 2026-09-16
---

# Issue25 熱保護相依成功仍要核對 worker 選取範圍

本次早期 QMD 回讀服務已加入 Requires/After 熱保護相依，但其名稱沒有命中 guard 的 issue25-v5-g*.service 選取條件。啟動相依只確保 guard 存活，不能保證新 worker 會被列入溫度控制。這與舊自動啟動鏈阻塞 guard 的開機排序事件是不同原因。

新 worker 採用符合選取條件的 g15 名稱，同時保留 Requires/After、AllowedCPUs 與正常 CPU 80% 配額。從同一使用者服務管理器核對實際 PID、有效屬性和 guard 日誌中的 active worker；不要把 systemctl 的 system 範圍 not-found 當成 user service 已停止。

本次停止名稱不符的回讀服務後，以受監控名稱續跑，保存既有輸出；後續 guard 日誌確認 active worker 且沒有觀察到過熱。guard 原有 GPU 動態配額與緊急降載仍優先，不把正常 CPU 80% 說成所有時刻固定值。未為測試刻意加熱或重開機，也不能由這次命名錯誤推論歷史重開機原因。
