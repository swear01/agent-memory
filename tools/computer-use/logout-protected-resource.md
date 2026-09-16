---
title: "靜態頁面 HTTP 200 不能證明仍然登入"
scope: tools/computer-use
status: active
updated: 2026-09-16
evidence_digest: 987f62ba52c8ea95c8ff92af3bd1b0d85bffd602d3f362f1868f8f33d947c754
---

# 靜態頁面 HTTP 200 不能證明仍然登入

助手曾用不受權限保護的靜態 HTML 驗證登出，後發現它即使未登入也回 200；改看站點登入頁特徵，貼出的結果在正式登出前後都顯示未登入。

使用該站可靠的受保護資源或明確登入狀態訊號驗證授權，不從靜態資源狀態碼推論 session。單一頁面字串也要先確認辨識語意。來源支持當次檢查方式修正，未證明所有裝置或其他 session 都已登出。
