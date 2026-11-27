---
title: Idle Spiral 100–1000×：原生時鐘隔離、宏步與跟著倍率的真實時間系統
scope: tools/openspeedy
tool: openspeedy
tags: [idle-spiral, harmony, unity-mono, steam, performance]
status: active
updated: 2026-11-28
---

適用於歷史案例 Idle Spiral 3.0.82 build 117（Unity Mono）、OpenSpeedy 3.3.8。遊戲已移除，以下保留判斷條件。補丁原始碼與原始驗證見 [封存 release](https://github.com/swear01/idle-spiral-openspeedy-patch/releases/tag/archive-2026-10-24) 和 [模擬預算](idle-spiral-simulation-budget.md)。

- **高倍率時原生等待變成忙等。** Native wait 會除以倍率後截斷，100× 時小於 100 ms 便可變成 0。案例停頓卡在 Steam 回呼。啟動即隔離原生時鐘／等待為 1×，託管模擬獨立讀倍率且只套一次。隔離後 100× 至 400× 長時測試未再停，Steam 內部機制仍未確認。
- **OpenSpeedy GUI 與程序實際狀態不同。** 自行載入 `speedpatch64.dll` 時，`SP_Install` 已開啟宿主，按原狀態決定是否立即 `SP_Disable(pid)`，保留先前注入的開關。倍率存在共享段，每程序開關在 `OpenSpeedy.<pid>`。關 GUI 仍可能持續，直接 `SP_SetSpeed` 也可能讓 GUI 顯示落後，測試後還原。
- **背景掉速需看模擬積壓與 FPS。** 在此版本，倍率逾 100 時 macro step 用 `ceil(倍率/100)` 同乘 gameDelta、baseDelta，步長至多 0.4 遊戲秒，Offline Boost 退回 1。40 遊戲秒的積壓上限同步放大，避免背景 20 FPS 截去每幀需要的步數。驗證失焦與實際推進，用 `ApplyResolvedCadence` 的實際讀回，短小的 resolver 可能已被 Mono inline。
- **日曆、寫檔與畫面刷新各用對應時鐘。** 日曆保留真實本地日期，點查存檔中舊的未來日期；真實午夜倒數和遊戲秒比較前先換算。完整寫檔按真實時間節流。自動戰鬥目標低於 Nuke 起始難度時，先查是否反覆重開造成 UI 抖動；檢查最終狀態與保存結果。細節見原始記錄。
- **驗收用未被 hook 的時鐘。** 用 `QueryUnbiasedInterruptTimePrecise` 量牆鐘，將實際狀態計數差、畫面和焦點對上；舊狀態檔可能停止更新。用不同 assembly name 避免 Mono 重用舊探針，正式組件換版須重開。若 Mono 的 `DateTime.Now/UtcNow` 和系統時鐘差固定偏移，檢查隔離生效前的啟用時段與取得時間的 API。

[原始調校、版本細節及各次量測](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/tools/openspeedy/idle-spiral-1000x-clock-isolation.md)
