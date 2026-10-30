---
title: Verto 尖刺／尖刺覆蓋方塊的慣例與實作
scope: projects/verto
status: verified
updated: 2026-10-31
tags: [Verto, spikes, hitbox, PhasePixels, PhaseController]
---
## 慣例（二手來源，非官方）

- 危險物的 hitbox 比圖小是通行做法；與圖同大或更大會讓玩家覺得不公平（itch.io／Godot 論壇討論、TV Tropes「Hitbox Dissonance」）。
- Celeste Classic：尖刺 hitbox 3px（右、下）或 2px（左、上），且只在玩家朝尖刺移動或速度為 0 時致命（CC Wiki「Spikes」）。完整版 Celeste 的細節沒有找到官方來源。
- 研究只搜到論壇層級資料；Super Meat Boy／VVVVVV／Geometry Dash 沒有可引用的來源。

## Verto 的實作（PR #41）

- 覆蓋尖刺的方塊本來就跟 R/B/G 相位走；問題在致命範圍是整格 + 0.5px。
- `PhaseController.Teeth(block)`：每面一條 1px 致命帶，角落各 `TeethInset = 2` px 安全。取代舊的 `Reach`；用到的地方：`CheckSpikes`、`Hazardous`、`SafeSpawn`、`PhaseCustomMap` 出生驗證、`PhaseSandbox` 的右側進入點搜尋。
- 尖端顏色改成奶白 `Tooth = f5f1e4`（舊 `Danger` 紅），普通尖刺與覆蓋方塊共用 `SpikePalette`。
- Swear Review 留下的低優先度 nit（未處理）：熱路徑每幀配置 `Block[]`；`PhaseCustomMap` 對尖刺平台只驗牙齒環而不驗本體。

## 驗證小技巧

- 用 `CheckHazards()` 直接設玩家座標驗證：玩家靠在左面、`y = blk.y-10.5` 擦過角落不死，`y = blk.y+1` 死。
- BuildWeb 在機器 load ~90 時 Play Mode 測試會卡死（Editor 主執行緒 hang），見 unity-hang-restart；殺掉 Editor、移開 `Temp/__Backupscenes` 後重跑即可。BuildWeb 會順手重烘 `Resources/Levels/*.png` 縮圖，記得 `git checkout` 還原。
