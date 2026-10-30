---
title: Verto 關卡設計：用無頭 PhaseController 模擬驗證路線與陷阱
scope: projects/verto
status: verified
updated: 2026-10-31
tags: [Verto, level-design, PhaseController, ChaosLevel, dotnet]
---
## 做法

設計產生器關卡（如 `Verification/ChaosLevel.py`）時，不必每改一次就開 Unity Play：

- 在 scratchpad 建一個 `net9.0` console 專案，`<Compile Include>` 直接引用 repo 的
  `PhaseController.cs`、`Laser.cs`、`Crystal.cs`、`Platform.cs`、`PhaseDemoRooms.cs`、`PhaseWorld.cs`、
  `PhaseCustomMap.cs`（加 Newtonsoft.Json）。這些檔案不依賴 UnityEngine。
- `PhaseCustomMap.Parse(json).Create(0)` 得到 controller，設 `exitRight = true`，每格
  `Step(new Controls{x=1, jump=..., jumpHeld=jump || (!p.grounded && p.vy<0)})`，跟
  `UmbrellaLiveInput.cs` 的鍵盤語意一致。
- 三種模式很好用：route（跳點清單 ±N px 平移都要通關）、explore（每個著地格分支「跳/不跳」，
  以 (x/4, phase, jumps%6) 去重，列出所有一直按右的可行路線）、windows（每一跳單獨掃可行起跳區間）。
  explore 對 160 格寬關卡約 2 分鐘；windows 每跳重播整條，較慢。
- repo 的 `ControllerTests.csproj` 目標是 net10.0；本機只有 .NET 9 SDK 時，`dotnet test` 無法跑，
  `-p:TargetFramework` 覆寫也沒用。改在 Unity Edit Mode 跑同一個測試，CI runner 有 .NET 10。

## 踩雷

- 不帶尖刺的彩色擋塊在跳躍弧線上會變成天花板：撞頭後跳躍變矮，原本該被雷射打到的「錯誤跳法」
  反而從光束下面穿過去。擋塊一律用尖刺方塊（`A`–`G`），碰到就死，也就不會改變弧線。
- 尖刺方塊自 PR #41 起只有齒的部分致命（每面 1 px 長條，角落內縮 2 px），分析檢查時把整格 +0.5 px
  當成致命，是偏保守的估計。
- 牆面側射的雷射要用整個區段「最高的地面」來決定高度，否則光束會從架高地板那一側的牆底下漏過去，
  打到在平台上走路的玩家。
- 解析式檢查（Python 弧線表）只負責擋塊擺放與快速 fail；最後一律用真正的 controller 重播確認。

## zsh 小坑

`git push origin $c:refs/heads/x` 在 zsh 會被當成 `$c` 的 `:r` 修飾符，refspec 變成亂碼。改用 `${c}:refs/...`。
另外，PR 內文引用的截圖若放在獨立的 assets 分支，合併後不要順手刪掉那個分支，否則圖片連結會失效。
