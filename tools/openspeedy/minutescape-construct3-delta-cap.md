---
title: Minutescape：OpenSpeedy 的兩種影格時鐘與 Construct 3 步進修正
scope: tools/openspeedy
status: active
updated: 2026-11-19
---

Steam Minutescape App 3327170、build 25433331，NW.js 0.100.1／Construct 3，實測 OpenSpeedy 3.3.20。原版 `package.nw` SHA-256 為 `BE9F864E19141B337B2F43CD1E4F54A7632BACDA5BB310E86850A986AAF43119`。

## 先區分注入與實速

- `speedpatch64.dll` 的完整路徑、`OpenSpeedy.<pid>` enable byte 和官方 `bridge64.exe GETSPEED` 分別證明注入、啟用與共享倍率，不直接證明玩法實速。以原生 perf、獨立 CLI Stopwatch／GetWallTime、實際 levelTime 及遊戲受擊 timeScale 交叉量測。`ISENABLED`／`STATUS` 不足以確認該程序狀態。
- OpenSpeedy NAME 每秒重套同名程序，單獨 DISABLE PID 可能立即被覆蓋。使用者要求以真實 OpenSpeedy 程序開關控制，單獨把倍率寫進遊戲不能代替。測不同倍率可用獨立的一套官方 bridge 與 DLL、限定 fixture PID，檢查正式倍率和 DLL 沒有被換掉。
- rAF 在 renderer-only 場景可能 1×，而 `performance.now()` 約 10×；六程序 NAME 場景兩個時鐘皆加速。用固定原始時間差當成中斷門檻會誤判，例如 25× 時 300 ms 真實畫格間隔是 7500 ms。舊版 6000 ms reset 會讓兩個 native clocks 同倍率的冷啟動反覆停在原點。
- Construct 由 rAF timestamp 計 `_MeasureDt`，丟棄大於 0.5 秒，將其餘值限制至 1/30 秒再乘原 `_timeScale`。Dodge #34 將 dt 累加到 `levelTime`，#82 寫 `recordTime`。修正用原始時間差一次、有限預算分多個 stock 步進、只繪製一次，不加大 stock dt 或改標準碰撞／慢動作。

## 排程與恢復

- 可信 WebAudio 時鐘可作真實時間交叉驗證，但也可能停頓後跳動，或和變速 perf 同斜率。保持有效校準，對中等慢畫格確認相容時鐘後做有界步進，真正長暫停不追回積壓。正式版在 3–15× 用 GUI 設定提供步進預算提示，只在正向、相容且 ≤500 ms 的原生差值使用，dt 仍由原生差值計算。
- 定時 FPS 與隱藏高頻 callback 的數字不能混比。約 0.59 ms 一次完整 Tick 時，100× 需要每秒 3,000 個 stock 1/30 秒 Tick，超過單線程能力。背景粒子可只在批次首步更新並合計 dt，關鍵事件、技能與碰撞仍逐步。隔離原生 100× 在固定回呼條件曾量到約 42× 有效值。
- Construct 啟動隱藏、visibility 切換與 Chromium 停止 rAF 要分別處理。`minutescape_background.js` 在桌面 NW.js vsync 背景用單一有界 timeout，完成才排下個，略渲染、尊重暫停與 context loss，切換時清時計及 `_dt1`。正式 10× 背景失焦可見實測約 9.95× 基礎模擬，沒有驗證真機最小化。
- OpenSpeedy `second instance exiting (window stays hidden)` 表示已有實例，第二次啟動不會喚回。先用托盤顯示或退出舊實例；補回磁碟 bridge 不會讓既有程序重生 child。遇 `WinError 225` 或 `Not Injected` 先分開確認 bridge、GUI、DLL 映射與遊戲，檔案恢復不代表注入恢復。

## 操作與挑戰

- F8 控自動巡走與技能，F9 控自動刷分。讀實際 `spr_box` 邊界與角色尺寸，不用固定 layout 尺寸；牆改動後先回到同側再轉角。F9 沿用原版 Play、結算、Upgrade 和存檔，布林用 `GetTypedValue()`。前端拖曳使用 canvas CSS 座標差轉成同一世界座標，避免 `Touch.SpeedAt(0)×dt` 兩時鐘倍率失配。
- 四技能以現裝備的 `singleLeft`／`singleRight` native held 觸發，死亡、換場或換裝清除計畫。Shrink 判定把原版碰撞先於縮放的兩步算入。Shockwave 要按住 2.5 遊戲秒才清半徑 400，Zap 要閃避。Shield 是方向扇形，原版預付 drain×0.4；從 native upfront 事件起計，避免合成 held 在首 0.4 秒多扣一次。手動按住時不接管。
- 零橘色先用 100 紫色購買 `shield give orange`，再用 Shield 擋彈；`shrink give orange` 本身需 30 橘色。藍色在 Dodge `levelTime >= 150` 且存活時生成。
- Stuck 原地做彈道預測，不能發 MoveTo。Broken L.A.B.（`noAbilities`）禁止技能，須清除殘留自動計畫。紫彈按原轉向限制逐步預測，藍彈按 native wait／當前速度判定階段。用原版 Play／Challenge／Simulate 與結算的本機 CLI marker，沒有任意 eval 或網路監聽；以 app Init 及真實 Tick 回報為準，完成即移除 marker。
- 正式封包 10× 驗收中，Stuck 到原版 5:00，Broken L.A.B. 最佳約 4:54 且未用技能。樣本受初始血量、彈幕與受擊影響。這些是正式驗收樣本，隔離的 85 個技能案例與原生 100× fixture 則證明各自的行為。

## 重建與還原

見 [Game Patch 保存與輸入版本](game-patch-cloud-backup.md)。最終 `package.nw` 經 ZIP 重建得到 `91EE7782B80199AB54660F6FA3D2E7544C06538A59075C0F53979F4852041C88`，`package.nw.before-late-wave` 是還原的上一補丁版，`package.nw.openspeedy-stock-direct` 是另外保存的原版。原始碼 ZIP SHA-256 為 `96EE5A78416AE216473001CFE289E5DA810C91555FD8C7A2EA10560DE4690DFF`。詳細版次、SID、指令、控制測試與證據可用 `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/openspeedy/minutescape-construct3-delta-cap.md` 查回。
