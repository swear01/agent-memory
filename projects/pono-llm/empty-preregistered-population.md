---
title: "預註冊族群為零要記 not-run 而非性能失敗"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 283201d23b096e7471ba0cbc60408f15ae440d5f4b0feb288eca8cf691369454
---

# 預註冊族群為零要記 not-run 而非性能失敗

Gate 4B0 的保存報告與使用者回應都記錄：固定 corpus 經語言與 branch cap 篩選後，natural eligible population 為零；H5a 未執行，development controls 的通過不等於 primary soundness 已跑。

保留原門檻與排除原因，不事後放寬或用 synthetic controls 補足自然族群。空分母不能作效能結論，in-process kernel 與含程序啟動的 solver 時間也不能直接相除稱 speedup。来源另有 leak-detection exclusion 與 Python bindings collection errors，沒有全部 canonical gate 乾淨通過。
