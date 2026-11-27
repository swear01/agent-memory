---
title: choke 低氧窒息視覺警示與 35% 門檻
scope: projects/choke
status: verified
updated: "2026-10-04T16:02:45+08:00"
---

- 使用者確認：氧氣低於 **35%** 就啟動視覺特效；35% 本身不啟動，34% 已有紅框與泛白。這取代先前視覺警示的 30% 門檻，適用 `Assets/Scenes/PlatformAirflowLab.unity` 的 `PlatformLabHUD`。
- 危險程度 `danger = Clamp01((35 - Oxygen) / 35)`；紅框與氧氣條開始脈動，隨缺氧由 0.9 Hz 加快至 1.6 Hz。氧氣條由亮紅轉暗紅；遊玩區淡粉白遮罩的 opacity 為 `danger² × 0.75`，越缺氧越亮。
- 氧氣歸零時，紅框固定全亮、泛白達 75% opacity，再接既有立繪滑入／滑出與 iris 死亡過場。撞傷等其他原因死亡、低氧通關都不顯示窒息泛白。補氧依目前氧氣減弱，恢復至 35% 以上清除；暫停凍結脈動，重生清除相位及警示。
- 保持「只留標示、不留說明」：不新增教程文字、死亡原因標籤或獨立憋氣條，也不縮放原地圖。
- 實作在 `Assets/Scripts/PlatformLabHUD.cs` 的 `Update`；`Assets/UI/PlatformAirflowLab/PlatformLab.uxml` 的 `suffocation-wash` 放在遊玩區、紅框下方，USS 設定淡粉白背景及初始 opacity 0。元素 `picking-mode="Ignore"`，不攔截操作。
- 35% 是此次視覺特效門檻；既有受傷立繪與心跳音仍為低於 30%，Dash 消耗／發動限制與 20% 以下的呼吸風力規則沿用現有設定。後續若要一起調整，需另有需求，不要全域替換所有 30。
- 渲染檢查另以暫時測試將 Panel Renderer 的 `panelSettings.targetTexture` 指向 1920×1080 RenderTexture，`ReadPixels`／`EncodeToPNG` 取得健康、低氧及嚴重缺氧的 HUD 畫面，完成後還原 target 並釋放資源。Batch Mode 的 `ScreenCapture.CaptureScreenshot` 在這次未產出檔案，不能只因呼叫成功就宣稱截圖成功；確認實際檔案與畫面。這是 HUD 渲染檢查，不能當作包含場景 Camera 的完整遊戲截圖。

- Editor 驗收涵蓋 35%／34% 門檻、暫停、補氧／重生、窒息死亡與非窒息死亡／通關；實際 WebGL 發布另驗。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/choke/hud-suffocation-warning.md)
