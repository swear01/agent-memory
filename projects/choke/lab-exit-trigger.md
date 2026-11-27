---
title: choke Lab 出口判定底邊對齊管道口
scope: projects/choke
status: verified
updated: "2026-10-04T13:48:47+08:00"
---

- 使用者要求逃脫偵測不要向下延伸；適用 `Assets/Scenes/PlatformAirflowLab.unity`，不能推論其他場景也有同樣設定。
- 根因：Exit 的 `BoxCollider2D` 原本以出口高度為中心、世界高度 2.4，底邊向下多出 1.2；玩家碰撞體碰到這個區域就會提早成功。
- 修正：底邊對齊 `exit.transform.position.y`，上緣保留 `exitY + 1.2`，寬度仍覆蓋整個管道口。中心向上移 0.6、世界高度縮為 1.2；本場景 localScale.y=0.3，因此 local offset.y=2、size.y=4。
- 三處需一致：場景 Collider、`Assets/Editor/PlatformAirflowLabSetup.cs` 的生成設定、`tools/CheckPlatformLab.cs` 的 bounds 檢查。只改場景會在重新生成後回到舊值；以 Unity API 修改並儲存場景。
- `PlatformAirflowLabTests` 驗證左右與中央在 `exitY - 0.5` 不成功，碰到出口則首個物理步成功。
- 固定物理時間與逐幀動畫的完成點不同。糖果卡住後的縮小銷毀測試應等物件實際消失，設 realtime deadline；不能只看到 `Time.fixedTime` 到點就認定動畫已結束。這次加入最多 1 秒的完成等待，保留原有銷毀斷言。
- 已實測 Editor。WebGL 與 itch.io 須按實際發布另外核對。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/choke/lab-exit-trigger.md)
