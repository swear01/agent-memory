---
title: "Subagent 模型顯示只採用實際 child 執行證據"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 2f5021cabf13753a34c42b304c57ca8f169bda6fdb5286bff455d61b9bf6d0f7
---

# Subagent 模型顯示只採用實際 child 執行證據

助手提議在缺 child 資料時顯示推定的父模型，使用者明確拒絕推定值，只要正確資訊。

將請求指定的 model、session metadata 與 child 回應實際回報的 model 分開。沒有 child 執行證據就不冒充實際模型；若過程換模型，要保留可證實的執行來源。

來源是研究與實作授權，沒有該修改完成證據。舊版 provider 支援表不當成目前能力；tool input.model 只證明請求，不證明執行。
