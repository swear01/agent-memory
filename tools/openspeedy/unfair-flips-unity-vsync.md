---
title: Unfair Flips 的 OpenSpeedy 加速需關閉 Unity 內部 VSync
scope: tools/openspeedy
status: active
updated: 2026-11-14
---

Steam 版 Unfair Flips（App ID 3925760，Unity 2022.3.62f2、D3D11）搭配 OpenSpeedy 3.3.8：遊戲程序已載入 speedpatch64.dll、每程序加速旗標為 true、共享倍率為 10.0，外部牆鐘取樣也確認被掛鉤的 QPC 約以 10× 推進，但預設 Ultra 畫質下，翻幣耗時仍沒有變化。單獨在 NVIDIA 為這款遊戲設 VSync Force Off 並重開後，從點擊至結算寫入存檔，1× 與 10× 各量一次都約 2.12 秒。

遊戲資產 globalgamemanagers 的 QualitySettings 顯示 Very Low、Low 的 vSyncCount 為 0，其他四個預設為 1。翻幣 coroutine 用 Unity Time.deltaTime 推進。透過 Steam 加入啟動選項 `-screen-quality Low`，再為新程序啟用 10×，同一 Steam 存檔兩次翻幣約 0.327、0.323 秒。當時 NVIDIA 個別遊戲覆寫仍保留，未單獨驗證拿掉驅動覆寫後的效果。量測包括點擊與存檔開銷，不可當作動畫本身達到精確 10× 的證明。

之後優先透過 Steam 開啟（直接執行 exe 的測試沒有初始化 Steam API，使用不同存檔路徑）。OpenSpeedy 3.3.8 的 PID 開關需針對重新啟動的遊戲程序檢查，必要時重新開啟。僅移除 Steam 的 `-screen-quality Low` 即可還原啟動畫質；沒有修改遊戲安裝檔。

[Unity Player 啟動參數](https://docs.unity3d.com/2022.3/Documentation/Manual/PlayerCommandLineArguments.html)；[Unity 對 VSync 與畫面時間戳如何影響 deltaTime 的說明](https://unity.com/blog/engine-platform/fixing-time-deltatime-in-unity-2020-2-for-smoother-gameplay)。
