---
title: "Wine 桌面 registry 寫入成功仍須量測視窗效果"
scope: tools/wine
status: active
updated: 2026-09-16
evidence_digest: dab729af04376604eac699efb4b1543bdebe98530eacd2947bac2f2c6679ce53
---

# Wine 桌面 registry 寫入成功仍須量測視窗效果

歷史助手回報已寫入 Wine 虛擬桌面尺寸，但主視窗與截圖尺寸沒有改變，Unity 仍寫回原本解析度；因此當次清晰度沒有改善。

將設定保存與應用程式採用設定分開驗證，對照重啟前後視窗邊界、截圖與程式內解析度。來源只證明該次未達效果，沒有確認根因，也不能推出所有 Wine 應用都忽略虛擬桌面設定。
