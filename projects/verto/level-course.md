---
title: Verto 關卡清單（PHASE jam 關卡清除後）
scope: projects/verto
status: verified
updated: 2026-10-31
tags: [Verto, levels, PhaseSandbox, level-select, thumbnails]
---
## 現行關卡（PR #47 合併後，main dbe3a01 起）

`PhaseSandbox.Levels` 決定順序、選關與連結（`LinkLevel()` 執行時重連，scene 裡的 leftScene/rightScene 只是顯示用）：

1. `PhaseDemo1`
2. `PhaseDemo1.1`
3. `PhaseDemo13`（level-a，三層 R→B→G 樓板 + 白雷射；已為 run/jump 重做，路線在 `Verification/Routes/PhaseDemo13.json`，`LevelRouteTests` 重播）
4. `PhaseDemo14`（level-b，左右交錯色雷射 + 紅綠柱；**仍無法用 run/jump 通關**，原路線靠爬柱，重做會失去 premise，所以保留並回報，線性進度會卡在第 4 關，選關可跳 5–7）
5. `PhaseDemo15`（chaos）
6. `PhaseUmbrella`
7. `PhaseGear`

已刪除：jam 時代的 `PhaseDemo2`–`8`、`PhaseCrystal1`–`3`、`PhaseDemo12`（需要 dash / climb / crystal / dash platform，現在遊戲只有 run、jump、wall slide、wall jump），以及 `CrystalRooms.json`、`DemoWebSmoke.cjs`。

## 注意

- 改關卡數量要同步：`BuildWeb.py` 的 SCENES 與 per-filter 測試數、`EditorBuildSettings`、`MapBridgeTests`/`MenuTests` 的選關編號、`Documentation/Web.md` 的 `SelectDemoRoom("1".."7")`。
- 只剩 7 關時直式 320×693 選關畫面在整數縮放下只填滿 0.5875，`PixelTests.MenusFitTheReferenceViews` 下限因此改成 0.55。
- 選關縮圖（PR #48）：`PhasePixels.Backdrop = false` 讓 `LevelThumbnails` 只畫地形/雷射在 ink 底色上，不畫霓虹背景。不能用透明底：`PhaseScreen.Image` 直接覆寫像素（含 alpha），會在面板上挖洞。
- `CreateImportedLevels` 會重寫全部 imported scene 與 URP global settings；只想重建一關時，事後 `git checkout` 其他 scene。
