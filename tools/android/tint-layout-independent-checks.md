---
title: "Tint 修好不能當成 layout 也已修好"
scope: tools/android
status: active
updated: 2026-09-16
evidence_digest: 7729e5dabf215b6ed51e371d0ffde0110c6f8e139e533ac2b6254180233f1a96
---

# Tint 修好不能當成 layout 也已修好

歷史助手在模擬器截圖看到圖示顏色恢復，卻仍有位置偏移與文字未渲染；讀取 view bounds 後又回報多個元素約縮小一半。

分別驗證顏色、位置、尺寸與文字。多個元素同時等比例改變時，先檢查共用縮放、density 與動畫狀態，再修改單一元素；這些只是診斷方向。來源沒有證實 XML 正確、動畫是根因，或後續修復成功。
