---
title: "Shell 變數存在不代表子程序會收到"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: a00a5a137a3349d69df8e071a86826eee7b9386b289b0aa6ccc334613ade70ae
---

# Shell 變數存在不代表子程序會收到

歷史助手回報 secrets 已被 source，shell 內 key 存在，但 child environment 沒有；手動 export 後 Pi 才看得到對應 provider。

以不顯示秘密值的存在性檢查，區分 shell assignment、export 與實際啟動環境。登入 terminal 的成功不代表當前工具程序或既有 session 已更新。來源明確說沒有修改檔案，因此永久設定仍未完成；auth ready 也不等於真實模型推論成功。
