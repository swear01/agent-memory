---
title: "Release 查詢失敗不能直接當作不存在"
scope: tools/github
status: active
updated: 2026-09-16
evidence_digest: d50d0496833bba921f813100b10e82f21fee434683e6a48c68a988d74269e77b
---

# Release 查詢失敗不能直接當作不存在

助手記錄 review 發現 gh release view 的網路錯誤被分成不存在，準備修正；相鄰 review availability 不是 release 狀態證據。

區分成功查得、明確 not-found 與 transport/auth 失敗。只有取得可靠不存在回應才走新建分支，避免暫時查詢失敗引發重複發布。

來源沒有這條錯誤分類的修後測試輸出，不從整體 merge 自述推定它已獨立驗證。
