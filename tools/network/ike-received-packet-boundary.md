---
title: "收到 IKE 封包只能證明該次觀測到的路徑"
scope: tools/network
status: active
updated: 2026-09-16
evidence_digest: c716489a02abdd1c5cce478e08d7d99e3321cac7ed1b0a06b46ca1b1caa62c9f
---

# 收到 IKE 封包只能證明該次觀測到的路徑

歷史助手回報防火牆收到一個來源的 UDP 4500，並記錄 Authentication method mismatch 與 Remote ID null，卻進一步稱 UDP 500、4500 與回程都正常。

按同次連線時間保存與對照 IKE log，分開判斷實際收到的封包、回程及認證階段。單一收包紀錄不足以证明全部網路路徑；認證錯誤應再核對雙方模式，不能直接當成所有 Windows 用戶端的通則。來源沒有憑證／EAP 配置或連線成功證據。
