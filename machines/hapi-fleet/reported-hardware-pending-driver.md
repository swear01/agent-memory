---
title: "待重啟的 driver 與實際硬體回報要分開記錄"
scope: machines/hapi-fleet
status: active
updated: 2026-09-16
evidence_digest: f9fa08794190bf0c8ae946704623cf105fbc20504cc400679569863874cbcdbe
---

# 待重啟的 driver 與實際硬體回報要分開記錄

來源逐機回報指出一張 GPU 的 Ti 型號與 Notes 表格不同，另一台的 nvidia-smi 損壞、安裝 driver 待重啟；助手準備據此修文件。

以每台實際可讀證據核對硬體與已載入 driver，缺失值保持待驗。不能把套件已安裝或其他主機的 CUDA 欄位補成該台 live 結果，也不能把修文件計畫寫成已更新。這份來源是歷史回報，不是目前 fleet 規格表。
