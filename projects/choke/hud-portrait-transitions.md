---
title: choke HUD 立繪直接切換與重疊淡化
scope: projects/choke
status: verified
updated: "2026-10-04T15:12:30+08:00"
---

- 使用者確認：`cough`、`hurt`、得分的 `love` 立繪切入及離開都直接換圖；`default`、`inhale`、`exhale` 才使用 0.2 秒重疊淡化，舊圖淡出時新圖同時淡入。
- `cough`／`hurt` 實際顯示後計時 0.3 秒；同一請求持續時回到 `default`，不反覆重播。其他表情取代、重生或 HUD 重新綁定會清除倒數。計時使用 unscaled time，不受玩法暫停影響。`love` 沿用得分後 0.8 秒玩法時間，再回到當下表情。
- 實作位於 `Assets/Scripts/PlatformLabHUD.cs` 的 `UpdatePortrait`；UXML 增加 `portrait-outgoing` 與原 `portrait` 疊放。普通表情反向切換交換兩張圖並延續當下透明度；第三張普通表情只保留最新請求，當前淡化完成後銜接。特殊立繪直接清除舊圖，避免殘影。
- 兩張圖保留各自原始比例，以 `default` 計算共同縮放倍率、共用中心與呼吸縮放週期。既有立繪中心偏差可接受；基礎尺寸、中心與死亡立繪中心測試容許 2 個 UI 像素，不要為了測試重裁圖或縮放地圖。
- HUD 的 0.4 秒死亡淡出仍獨立套用；`IsDeathSequencePlaying` 保持立繪隱藏直到過場結束。重生清除雙圖轉場與表情倒數，保留既有滑入／滑出及 iris 死亡過場。
- 驗證：Unity 6000.6.4f1 的 `PlatformAirflowLabTests`（EditMode，`Fast;Integration`）5/5 通過，約 24.14 秒。涵蓋雙圖同時可見、透明度互補、快速反向切換、特殊立繪直接切換、暫停時的 0.3 秒倒數、恢復呼吸取消倒數，以及死亡／勝利回歸。
- 已合併並推至 main `ed93c3b`，該提交 Unity Git safety CI 成功，原專案已同步。這是當時的原始專案／Editor 驗證，沒有重建或發布 WebGL；後續 main 持續變動，重用時需確認目前程式及測試。
- 測試教訓：咳嗽仍在作用時會優先於吸氣立繪；驗證受傷後恢復吸氣前需讓咳嗽結束。此測試框架使用既有 `PhysicsSeconds` 推進固定物理時間，另以 realtime deadline 等待逐幀 UI 完成，不以時間到點代替畫面斷言。
