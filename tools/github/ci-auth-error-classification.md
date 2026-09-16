---
title: "CI 的 gh 認證失敗不能藏成找不到 release"
scope: tools/github
status: active
updated: 2026-09-16
evidence_digest: 0f24ace1a4f17b1de7c66a605237e06f6863433a6c18941a26071d2862632b66
---

# CI 的 gh 認證失敗不能藏成找不到 release

歷史助手遇到本機正常、CI 回報 no previous stable release found，後將差異定位到 gh 缺少明確 token 設定，並發現原始錯誤被通用訊息遮蔽。

依 CI 的既有安全設定提供所需認證，保留可辨識的認證與查詢錯誤分類；不能把查詢未成功當成真的沒有 release。診斷可保留必要 stderr，但不得輸出 token。來源沒有修正後 workflow 通過的證據，也不表示所有 runner 都缺少認證。
