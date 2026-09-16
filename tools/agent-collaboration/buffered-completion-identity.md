---
title: "完成通知先對應目前 run 身分"
scope: tools/agent-collaboration
status: active
updated: 2026-09-16
evidence_digest: 7c66f935bae14daab247ed99f430a1cfabea0eb2351ea03423c013863078a6c2
---

# 完成通知先對應目前 run 身分

歷史助手發現收到的是上一趟 pipeline 的緩衝摘要；當前 run 的 process 與輸出仍顯示正在執行，不能把前一趟結果當成目前完成。

對完成事件核對 run ID、程序身分、開始時間與該 run 的終結產物，再更新狀態或啟動後續階段。PID 或 elapsed 只能輔助比對，不能單獨證明輸出歸屬。來源只支持這次誤認與更正，沒有監控器修補結果。
