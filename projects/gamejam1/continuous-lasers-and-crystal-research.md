---
title: PHASE continuous lasers and jump-focused crystal research
scope: projects/gamejam1
status: lasers-merged-crystals-proposed
updated: 2026-10-18
tags: [PHASE, Gamejam1, laser, crystal, jump, dash, level-design]
---

## 已合併的雷射

- PR #24 implementation `5a47a17c45b1c120a4cccbcb3b98b2722e1338c4` merged into main as `4226e19d510cb438ef6a4512f3035cc9fecd6570`. User explicitly authorized merging without waiting for unavailable bot review. The latest-head controller CI passed; local main was synchronized and the task branch/worktree removed after preserving verification artifacts.
- `Laser` / `PhaseLaser` use an8px emitter,4px lethal beam and four cardinal directions. Seven masks R1/B2/G4/Y5/M3/C6/W7 follow current player phase; white is always active. Beams are continuous and have no timed firing cycle. Safe beams remain translucent. Current solid terrain, dash platforms and both climb-gear parts truncate beams; ghost/deferred terrain and decorative racks do not.
- Rendering and swept collision share `Laser.Beam`; `CheckHazards(before)` covers jump phase changes, dash correction and mechanism carry. Decoration cannot hide a lethal beam. DemoRooms JSON, builder and all8 saved scenes contain emitters.
- Verified before merge:51 .NET tests,65 Editor tests,3 Play Mode tests; the final rendering-order fix reran13 PixelTests. All8 live keyboard routes and7 checkpoint transitions passed with0 deaths. Live probe checked climbing-driven white cover, immediate exposure after cover moves, blue activation on keyboard jump and respawn.
- Console contained a UnityEditor.Search.SearchDatabase indexing exception; do not claim an empty Console. PR #24 itself had no WebGL build/public deployment. Preserve this release boundary and recheck live state before any online claim.

## 早期研究方向（2026-10-18整合已實作水晶，見下方）

- User wants a dense multi-color laser maze with a route to discover, primarily through jumping and phase changes. They requested several independent Claude design consultations and investigation of whether removing dash improves authoring/play; existing dash must remain for now. No removal decision has been made.
- Research interprets the two neutral-white shapes as ROUND = one extra airborne jump, SQUARE = one dash refill. Pickup does not itself change phase; spending an extra jump would cycle R→B→G→R. These were research proposals at that point; the verified colored-crystal implementation is recorded below.
- Current ground/coyote/wall jumps cycle phase; climb/fall/dash preserve it. No dash invulnerability. `ActivatePlatform` is triggered by dash, so deleting dash also requires replacing the ferry trigger and revisiting dependent rooms. Gear-linked motion is driven by actual climbing and does not require dash.
- Every new air-jump design must inspect the transition instant: newly active laser, disappearing landing, uncovered white beam. Existing newly solid occupied terrain already defers through `blocked`; do not invent a push-out or prohibit-shift rule without checking that behavior.

## 控制器研究量測，非人類試玩

- Production source at merged main,60Hz, .NET10.0.400: a fully held ordinary jump rises27native pixels and moves57px from rest or62px from a90px/s running start before same-height landing. A one-frame tap rises7px. These are displacement measurements, not comfortable gap limits.
- A separate gap search with player width and coyote allowed found single-jump witnesses across73px at equal height,64px at16px rise and53px at24px rise. Those witnesses include narrow landing margins; never equate62px displacement with a hard maximum gap or call a72px gap crystal-required.
- An8-emitter research layout using the existing staircase geometry passed3 jumps/0 dashes/0 deaths. Among651 fixed-route variants inserting one extra dash,387 reached the same landing with0 deaths, and none used fewer than3 successful jumps. This is a bounded witness study, not exhaustive reachability, uniqueness or human-balance proof.
- Temporary direct calls to the existing `Jump` motion were used only to estimate proposed air-jump trajectories. They do not implement crystal pickup, fresh-press input, consumption/respawn, Unity scene behavior or product delivery.

Source: `<project-root>/Assets/Phase/Runtime/PhaseController.cs`, `Laser.cs`, `PhaseDemoRooms.cs`; artifacts under `<user-documents>/Codex/2026-10-17/gamejam1-lasers/` and `phase-laser-level-research/`.

## 七色水晶與十一關整合（2026-10-18）

- 新的直接使用者指令要求將原PR27與所有待合併工作整合main，取代該參考聊天先前的「保留PR等待審查」。來源PR27是5d1476b54aeee161b11ce8974370412e2e897b36，中文Wiki PR26是4b15af446e07ed3f9c16e766f54cc459efea48e4；整合分支從main06ad5de建立，保留兩者完整parent history。
- 圓水晶提供最多一個fresh-press空中補跳，拾取不換色，使用跳躍才換色；方水晶無條件dash=1，不要求先耗掉、不改色、不取消正在衝刺。兩種形狀支援七個mask，只有相位相符才能拾取，錯色ghost；實際落地或reset後rearm，落地清除補跳。危險檢查先於拾取。
- 新三關PhaseCrystal1–3接在PhaseDemo8後，選關9–11：空中換色、十雷射的三色光網、保色雙衝刺接力。前三關仍保留原mechanics；沒有移除dash。Round/dash兩個Prefab加入PHASE/Add，合計七個mechanism Prefabs；所有十一關生成場景透過resident Editor重建並保存Prefab overrides。
- 合併PR27舊single-mover介面時不可退回mover欄位、hopMover/dashRide booleans或舊逐台carry。保留movers集合、identity與共同pixel substeps，將CollectCrystals(before)接到同步carry/push每個substep的危險與crush檢查後。既有pickup測試改成兩台實際同時移動，確認載人拾取與另一台行程均保留。
- 實測140resident Unity CLI/Pipeline cases：134Edit+6Play；108.NET、7Python、Web/map guards通過。所有11條live keyboard routes/10次checkpoint transitions通過、0死亡，advancing frame/controller time與source=screen Game View確認，fresh Console0errors0warnings。原五個immutable fixture與手工PhaseDemo1.1逐byte不變。
- 合併後GridGeometryTests實際6case，不是把兩分支舊counts相加；首次預期7導致strict guard失敗，保留結果後修正BuildWeb.py/文件為6。使用實際測試報告核計140，不忽略countguard。冷啟動EDITOR_NOT_READY等待後再試，ShaderGraph GUID首次compile錯誤由Unity API Updater自動修正；原log保留，沒有batch fallback。
- 中文Wiki整合為十一關、補跳/水晶、multi-platform與parent-relativePrefab說明，並加六張新關live PNG；原八關圖片仍標示e4332a2來源。桌面/手機、file/HTTP、無JavaScript、圖片/anchors/11個folded routes皆通過，Impeccable distill與detector[]。
- 本次僅整合Git/main，未重新build或部署公開Web；先前PR27獨立built-Web證據不能當成此整合版Web證據。人類平衡與窮舉捷徑未測。證據在<user-documents>/Codex/2026-10-18/gamejam1-integrate-all/，94sourcefiles與完整TestResults已byte-verified封存。
- 整合PR30 head1a8b6bc98c411f703352ef4712d4fe14702401d0，controller CI39054436086通過，合併main為dab002ec4a2015df772dd40815c1fafd7c5317b0。GitHub同步將PR27標MERGED（05117ff225c6d33a3a723c75455629af6e7fcdb0）、PR26標MERGED（1a8b6bc98c411f703352ef4712d4fe14702401d0），沒有open PR。人工review授權下整合，未宣稱bot pass；source heads均為main ancestors。
