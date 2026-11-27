---
title: PHASE continuous lasers and jump-focused crystal research
scope: projects/gamejam1
status: lasers-merged-crystals-proposed
updated: 2026-10-18
tags: [PHASE, Gamejam1, laser, crystal, jump, dash, level-design]
---

- `Laser`／`PhaseLaser` 用 8px emitter、4px 連續致命 beam 與四正交方向，mask R1/B2/G4/Y5/M3/C6/W7；白色恆亮，安全光半透明。只有實體地形、dash 平台、雙 climb-gear 零件截光。Ghost、deferred 地形或裝飾不擋光或遮住危險。Render 和 swept collision 共用 `Laser.Beam`，`CheckHazards(before)` 檢查跳躍換相、dash correction 與 carry。
- 圓水晶提供最多一次 fresh-press 補跳，拾取保持相位，使用時換色，落地清空。方水晶無條件 dash=1，不換相、不打斷既有 dash。相符的七種 mask 才拾取，錯色 ghost，落地/reset rearm。使用者要求保留 dash。目前 dash 無無敵，`ActivatePlatform` 依 dash。
- 多 movers 要維持集合、identity 與共同 pixel substeps；不要退回單 mover 或 hopMover/dashRide bool。每個 carry/push substep 在 hazard、crush 檢查後 `CollectCrystals(before)`。測兩 mover 實際同時移動、載人拾取和另一台行程。
- 60 Hz 普通跳滿按上升 27 native px，水平靜止 57／助跑 62，單幀 tap 上升 7。含角色寬與 coyote time 的單跳存在 73px 同高、64px +16 高、53px +24 高的 witness。位移不等於可跨上限，72px 有機會不用水晶。自動 route/witness 也不能證明人類平衡或所有捷徑。
- Build count guard 查實際 GridGeometryTests（原整合為 6），按報告同步 BuildWeb.py／文件。Unity CLI 冷啟動 `EDITOR_NOT_READY` 先等再試，ShaderGraph GUID 初編譯可由 Unity API Updater 修復並保留 log。整合檢查含 resident Editor/Game View；正式 Web build 和發布需另驗。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/gamejam1/continuous-lasers-and-crystal-research.md)
