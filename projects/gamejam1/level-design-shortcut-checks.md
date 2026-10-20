---
title: PHASE 關卡設計與捷徑檢查（第 1 章顏色草稿）
scope: projects/gamejam1
status: active
updated: 2026-10-20
tags: [PHASE, Gamejam1, level-design, shortcut, spike, dash, route, simulator]
---

# PHASE 關卡設計與捷徑檢查

第 1 章草稿（PR #42，`Assets/Scenes/Drafts/Color*.unity`）驗證過的經驗。

## 彩色尖刺的起跳規則

`Jump()` 先 `Shift()` 換色再立刻 `CheckInteractions()`，站在尖刺根部時用「跳完的新顏色」判定。
所以紅刺（mask 1）只能在藍色時起跳（跳完變綠仍安全），在綠色時起跳會變紅當場死亡；
雙色尖刺（mask 3/5/6）只有一種安全色而且下一色必定致命，只能走下去，不能從上面起跳。
這條規則可以讓地板謎題每一步只剩一種顏色可行，也是衝刺打不穿 1-4 的原因。

## 移動可達範圍（設計捷徑封鎖時用）

- 一般跳：上升 27 px，同高水平 62 px；跳＋水平衝刺約 104 px；跳＋上衝刺 73 px。
- 地面衝刺接跳（hyper）水平 325 px/s。
- 蹬牆跳只要離牆 3 px 內就會觸發，不必貼牆；禁止區域測試要把牆旁那一欄一起禁掉，不然會誤判「這段牆不需要」。
- 上方 5 列內的彩色地板接縫、白色平台邊緣，都能從下面抓住爬上去（顏色對的時候）。
- 層距 7 格（56 px）的三層房間擋不住上衝刺；教學房要防衝刺得用更高的層距（捲動房間）。
- 白色平台是任意換色的「跳點」，放在後段平台的衝刺範圍內會讓顏色限制失效。

## 驗證工具與坑

- 用 dotnet 主控台把正式 `PhaseController.cs`、`Laser.cs`、`Crystal.cs` 編進來做分段最佳優先搜尋（server GC＋平行展開），
  比 Editor 內搜尋快約 10 倍；整房 BFS 會爆，要用分段目標和禁止區域。
- 地圖一定要從 scene 匯出，不要手抄：手抄 `PhaseDemo1.1` 時右牆錯一欄，搜出一條實際不存在的捷徑。
  從 scene 重新匯出後找到真的捷徑（沿底部衝刺從右牆下方出去，135 格），並用路線工具與實機鍵盤重播確認。
- 掛在 `PhaseSandbox` 上的重播協程，在出口載入 Build Settings 裡的下一個 scene 時會被銷毀，`finally` 不會執行；
  要在 `SceneManager.activeSceneChanged` 裡還原 `InputSystem.settings`、移除虛擬鍵盤並回報結果。
- 正向進房（`checkpointOnEntry`）會把玩家放到出生點並重設為紅色，房間出入口高度只影響切換畫面是否連貫。
