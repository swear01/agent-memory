---
title: "Windows 上 GameMaker Studio 1.x 的高速遊玩測試"
scope: tools/games
status: active
updated: 2026-11-19
---

# Windows 上 GameMaker Studio 1.x 的高速遊玩測試

在《Minimalism》的本機 GameMaker Studio 1.x Windows runner，`room_speed = 300`、關閉 VSync 並降低繪製頻率後，連續 16 個遊戲步仍要約 0.23–0.26 秒。該 runner 的 `fps_real` 曾顯示超過 2,000，與牆鐘時間量得的更新率不同。

在測試副本的 `options.ini` 使用以下組合，保留 `room_speed = 300`，連續 128 個遊戲步實測約 0.40–0.44 秒，接近原本 60 步／秒的五倍速。這次只驗證了組合，沒有分別驗證兩項設定的效果，也沒有驗證其他 GameMaker 版本。用遊戲步數除以外部量得的經過秒數，確認實際速度。

```ini
[Windows]
AlternateSyncMethod=1
SleepMargin=15
```

平台遊戲的路徑規劃器即使只和原生 runner 差一兩個像素，也可能把擦過尖刺的路判成安全。每批控制指令從遊戲回報的最新位置、速度與按鍵狀態重新模擬。如果後續指令已預測會碰撞，就從該回報狀態重規劃。不要依賴位置偏差超過固定門檻才重算。
