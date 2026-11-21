---
title: Outhold 的可見 CLI 作戰、十倍速驗證與成就門檻
scope: projects/outhold
status: active
updated: 2026-11-21
tags: [outhold, godot, openspeedy, cli, steam]
---

## 同一個遊戲程序操作

CLI bridge 載入原本的 Godot `user://` 存檔槽，呼叫原生關卡路由和建塔、升級、開波、提早開波的操作，讓同一個可見 GUI 即時顯示。用常駐 bridge 接收下一份關卡計畫。等待原生結算完成，再讀存檔的對應完成旗標及 Steam 成就狀態。Endless 達標後可用關卡內原生選單的 `End Run` 結算。已驗證完成 20 波後從選單結束，遊戲照常記錄波數、完成旗標與首次獎勵。操作限制見[自動玩遊戲的操作與通關偏好](../../global/preferences/gameplay-automation.md)。

將 OpenSpeedy 10× 套到執行可見 GUI 的 bridge PID。用 `Time.get_ticks_usec()` 相對 `Time.get_unix_time_from_system()` 的增量核對程序時鐘，實測約 9.97–10.02×。程序換 PID 時重新確認。

另取樣 `Engine.get_physics_frames()` 和 `Engine.get_process_frames()` 相對牆鐘的增量。執行中將每個渲染影格的物理步進上限由 8 提到 24，關閉 VSync 與 FPS 上限，使用 1280×720 視窗。固定步長仍是每遊戲秒 60 個 physics ticks，`Engine.time_scale` 為 1.0。輕負載量到每真實秒約 594–598 個步進。Prestige 9 後段敵人密集時降至約 200，因此程序時鐘 10× 時，實際戰鬥速度仍隨渲染負載變動。

## Ice Artifact 卡在第六波

第六波是 50 個 Runner 與 25 個 Elite Runner 同時進場。已通關的配置先在冰屬性中央格放 Arrow `(0,0)`、Pulse `(0,-1)` 和 `(0,1)`、Arrow `(1,0)`，後續加 Arrow `(-1,0)` 與 `(1,1)`。開波前有四塔時，在血量充足且存活與待刷敵人數可控的波次提早開波，拿原生寶石及 Charge。提高 Arrow 射速，補 `SLOW_TOWER_DAMAGE_LINK`、`SLOW_TOWER_DAMAGE_LINK_INCREASE` 與 `SLOW_TOWER_DAMAGE_LINK_SHARE_SOURCES`，並補齊 `CORE_HEALTH_3` 的核心擊殺獎勵前置。這組調整已用原存檔完成 10 波。

`ARROW_TOWER_FIRE_RATE_2` 與滿級 `ARROW_TOWER_FIRE_RATE_INCREASE_DAMAGE_DECREASE` 可把重箭的 4.17 秒射擊間隔降到 3.13 秒。後者由 `ARROW_TOWER_MULTISHOT` 解鎖，也是 `ARROW_TOWER_POISON` 的前置。需要毒時，依序買 `ARROW_TOWER_POISON`、`ARROW_TOWER_MORE_POISON`、`ARROW_TOWER_CRIT_POISON`，再買相依升級。Ice 的通關紀錄驗證了射速與提早開波調整。

## Prestige 9 的蜈蚣首領

已通關的配置以 `(5,-1)` 的天然 Fire Lightning 清群，升到 9 級，搭配 `(3,-1)` 的天然 Wind Fire、`(1,0)` 的 Light Pulse 和 Bank。`(1,-2)` 放 Fire Arrow 並升到 8 級。蜈蚣的節段共用 4,000 萬血量，用連鎖與範圍火傷打多個節段。這局全程靠塔輸出。

## Steam 成就

- `Master` 由原生關卡結算檢查。普通關 10 關，Time Trial、Specialization、Token Limit、Prestige、Endless 各 9 關，另有 5 個 Artifact 關卡。Endless 第 1 關要完成 10 波，第 2–9 關要完成 20 波。原生一般與無限制模式共用完成旗標，第 9 關直接用一般 Endless。
- `MAXED!` 由活躍中的 Upgrade Tree 檢查。普通關全通、目前配置有至少 5 個 Artifact，以及非里程碑升級已花 Basic 68,715、Advanced 53,600、Star 19、Time 36、Challenge 36。五種第一份 Artifact 即符合數量。透過升級介面的原生購買觸發判定並等候更新。
- 普通關與 Prestige 提供全部 19 Star。九關 Time Trial、Token Limit、Endless 的首次獎勵分別合計 9、9、18 Time。九關 Specialization、Token Limit、Endless 的首次獎勵分別合計 9、18、9 Challenge。

- MAXED! 的購買清單直接取原生 Upgrade Tree 的完整節點。`SaveSystem.save_game()` 只把目前配置中公開狀態達 `REVEALED` 的升級寫進存檔，單次讀檔會漏掉前置尚未買到的後續節點。這次補買並重讀後，清單從 175 增至 183、最後達 194 個節點。五類代幣支出全部符合原生門檻後，Steam 驗證為 15/15，含原生 MAXED!。
