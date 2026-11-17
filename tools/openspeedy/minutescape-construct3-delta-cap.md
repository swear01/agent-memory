---
title: Minutescape 的 OpenSpeedy 加速受 Construct 3 單幀時間上限限制
scope: tools/openspeedy
status: active
updated: 2026-11-18
---

Steam Minutescape（App 3327170，build 25433331，NW.js／Construct 3）搭配 OpenSpeedy 3.3.20：六個遊戲程序（包括 renderer）都有載入 speedpatch64.dll，且唯讀開啟每個 OpenSpeedy.<pid> mapping 得到旗標 byte 1。從 OpenSpeedy 安裝目錄以重導 stdout 的方式執行官方 bridge64.exe GETSPEED 回傳 10.000000，與保存設定一致。這些只證明掛鉤狀態和設定倍率，不能證明玩法倍率。

安裝版 package.nw 的 data.json project[37] 是 vsync。scripts/c3runtime.js 將 requestAnimationFrame 的 timestamp 送給 Tick 和 _MeasureDt。非影片匯出模式將相鄰時間戳差換算成秒；超過 0.5 秒的差值直接歸零，其餘夾在 0 與 1/30 秒之間，再乘遊戲自己的 _timeScale。Dodge 事件 #34 將 System.Exps.dt 累加到 levelTime，#82 把 levelTime 寫入 recordTime，因此限制會影響關卡計時。遊戲也用自己的 timescale 做閃避慢動作。

以安裝版 _MeasureDt 的計算段做離線 timestamp 重播，固定遊戲 timescale=1：每真實秒 60 個 tick 時，1× clock 推進 1 遊戲秒，10× clock 只推進 2 遊戲秒；120／144 tick 時 10× 分別推進 4／4.8 遊戲秒。若是每秒 60 個 tick 且時間戳差放大為 40×，會因單次大於 0.5 秒而全部歸零。這些是注入合成時間戳得到的程式結果；尚未量到 live renderer 的 rAF clock 或關卡真實倍率，不可把顯示器刷新率當成實測 tick 數。

若後續做相容修正，需先量測 live renderer 的 rAF timestamp 與關卡時間；必須保留遊戲原有閃避 timescale，並避免同時對原生 clock 和玩法重複套倍率。這次沒有修改遊戲、存檔或 OpenSpeedy 設定。
