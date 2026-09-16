---
title: "下一次取訊息前先完成待送佇列"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 47367bd0dee1b6fec5aa69b3bba5718bea676b925dcaa93f3958f83e553e522d
---

# 下一次取訊息前先完成待送佇列

保留的程式 diff 在 Claude nextMessage 取走訊息前加入 await messageQueue.flush()，用以處理延後排程的送出佇列與下一輪讀取之間的順序問題；來源含 commit 回報。

把需要的先後關係放在消費佇列的共同入口，並以實際 callback 與 await 路徑驗證。不要用微任務永遠先於宏任務的簡化說法取代排程分析。來源能支持這個修改及其意圖，沒有端到端成功或 PR 合併證據。
