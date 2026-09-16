---
title: "Picker 的 flat 分支可能保留原本長清單症狀"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 0728dfe5fdf6f667066498f01942b7ea1874eb40d7ca7071f984c1dbce610208
---

# Picker 的 flat 分支可能保留原本長清單症狀

歷史助手宣稱 Cursor picker 改成 raw base／variant split 且測試全綠，使用者仍看到很長且未排序的清單；助手才懷疑沒有 multi-variant 時保留的 flat 分支。

用實際 catalog payload 檢查每條呈現分支，以及它是否重現使用者原先症狀。UI flat fallback 與 CLI set_model fallback 是不同路徑，不能混稱都已移除。來源只到根因假設，沒有最終修復或畫面驗證。
