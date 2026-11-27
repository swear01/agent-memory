---
title: Max Manos 的 OpenSpeedy 加速受 VSync 限制
scope: tools/openspeedy
status: active
updated: 2026-11-26
---

## 倍率顯示正確，遊戲速度仍不變

Windows Steam 版 Max Manos／OpenSpeedy 3.3.8 驗證。先確認目標程序的 DLL、enable flag 和實際共享倍率，再查 `optVSync`。關 VSync 後使用者確認速度有變。全螢幕和視窗模式也要分開測；當時 Alt+Enter 往返後，VSync Off 仍約 60 Step/s，重開才恢復。先讓視窗與焦點恢復，再確認讀的是實際模式。未證實 DWM 或驅動哪一層造成限制。

- `data.win` 的 GEN8 基準 60 FPS，`objCTRLgame` Step 才扣 `runTimer`，`runTimer mod 60` 控秒邊界；10× 約需 600 Step/s。用獨立外部牆鐘與幀計數交叉量測，`fps`、`fps_real`、`delta_time` 可能跟著被掛鉤的時鐘。`PresentMon` 回 `failed to start trace session: access denied` 時純讀引擎計數可繼續定位。
- 輕負載約 585–597 Step/s；同機以 600 個物件的合成戰局只約 200 Step/s。提高到 100× 並未顯示額外吞吐量。關 Show Numbers 或調 SleepMargin 的小差異在隨機位置及升級變動下無法支持穩定收益。重負載最忙執行緒接近滿載，但沒有函式級 profiler，也未重現使用者原本的後期連鎖。
- 存檔會優先透過 `steam_file_exists`／`steam_file_read`，也會寫 Steam Remote Storage。測前備份 LocalAppData 和 Steam `userdata/<account>/<appid>/remote/maxmanos.ini` 兩份，測後均還原、比對 SHA。

## Draw 跳幀的驗證

FingerGun 的瞄準 `crosshairX/Y/Dir` 原在 `objCTRLgame` Draw GUI End（Draw_75）。要試跳幀，先將原計算移到 End Step，只對未暫停戰鬥跳 Draw，保留選單與等待畫面。相同合成戰局量到每幀、每 5 步、每 10 步繪圖分別約 199、251、261 Step/s。驗證步進和扣時一致、瞄準仍更新、暫停不扣時、結算與存檔正常。這是部分收益，沒有達 10×，也未證明所有玩法等價。

公開 [Game Patch](game-patch-cloud-backup.md) 的 Max-Manos 目錄用 `Build-MaxManosPatch.ps1` 與 UndertaleModCli 0.9.2.0，從符合 SHA 的原版離線重建，與歷史測試版 SHA 相同。輸入版本不符、輸出已存在會停。歷史測完已還原原版與兩份存檔。逐組樣本、封裝檢查和交付見 `git show 2505b5e:tools/openspeedy/max-manos-vsync.md`。
