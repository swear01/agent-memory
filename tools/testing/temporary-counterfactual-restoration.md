---
title: "反事實測試的臨時 patch 必須驗證還原"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 4b5a583d246fdec0e13198eb3608d80a32cab205451799ad657aca6746b91ef2
---

# 反事實測試的臨時 patch 必須驗證還原

歷史助手為測舊 launcher 行為暫時回改程式，卻因 trap 相對路徑錯誤未還原；手動復原後自述取得修復版 pass、舊版 timeout、復原版再 pass。

用穩定目標路徑與保存的原始內容管理臨時修改，結束後核對 bytes 與測試狀態；trap 註冊不等於還原發生。來源僅涵蓋 mock app-server 的 launcher 回歸，不是真實 app-server 端到端，也未獨立重跑。
