---
title: PHASE continuous lasers and jump-focused crystal research
scope: projects/gamejam1
status: lasers-merged-crystals-proposed
updated: 2026-10-17
tags: [PHASE, Gamejam1, laser, crystal, jump, dash, level-design]
---

## 已合併的雷射

- PR #24 implementation `5a47a17c45b1c120a4cccbcb3b98b2722e1338c4` merged into main as `4226e19d510cb438ef6a4512f3035cc9fecd6570`. User explicitly authorized merging without waiting for unavailable bot review. The latest-head controller CI passed; local main was synchronized and the task branch/worktree removed after preserving verification artifacts.
- `Laser` / `PhaseLaser` use an8px emitter,4px lethal beam and four cardinal directions. Seven masks R1/B2/G4/Y5/M3/C6/W7 follow current player phase; white is always active. Beams are continuous and have no timed firing cycle. Safe beams remain translucent. Current solid terrain, dash platforms and both climb-gear parts truncate beams; ghost/deferred terrain and decorative racks do not.
- Rendering and swept collision share `Laser.Beam`; `CheckHazards(before)` covers jump phase changes, dash correction and mechanism carry. Decoration cannot hide a lethal beam. DemoRooms JSON, builder and all8 saved scenes contain emitters.
- Verified before merge:51 .NET tests,65 Editor tests,3 Play Mode tests; the final rendering-order fix reran13 PixelTests. All8 live keyboard routes and7 checkpoint transitions passed with0 deaths. Live probe checked climbing-driven white cover, immediate exposure after cover moves, blue activation on keyboard jump and respawn.
- Console contained a UnityEditor.Search.SearchDatabase indexing exception; do not claim an empty Console. PR #24 itself had no WebGL build/public deployment. Preserve this release boundary and recheck live state before any online claim.

## 使用者目前的方向，尚未實作

- User wants a dense multi-color laser maze with a route to discover, primarily through jumping and phase changes. They requested several independent Claude design consultations and investigation of whether removing dash improves authoring/play; existing dash must remain for now. No removal decision has been made.
- Research interprets the two neutral-white shapes as ROUND = one extra airborne jump, SQUARE = one dash refill. Pickup does not itself change phase; spending an extra jump would cycle R→B→G→R. Crystals and these lifecycle rules are proposals, not shipped behavior.
- Current ground/coyote/wall jumps cycle phase; climb/fall/dash preserve it. No dash invulnerability. `ActivatePlatform` is triggered by dash, so deleting dash also requires replacing the ferry trigger and revisiting dependent rooms. Gear-linked motion is driven by actual climbing and does not require dash.
- Every new air-jump design must inspect the transition instant: newly active laser, disappearing landing, uncovered white beam. Existing newly solid occupied terrain already defers through `blocked`; do not invent a push-out or prohibit-shift rule without checking that behavior.

## 控制器研究量測，非人類試玩

- Production source at merged main,60Hz, .NET10.0.400: a fully held ordinary jump rises27native pixels and moves57px from rest or62px from a90px/s running start before same-height landing. A one-frame tap rises7px. These are displacement measurements, not comfortable gap limits.
- A separate gap search with player width and coyote allowed found single-jump witnesses across73px at equal height,64px at16px rise and53px at24px rise. Those witnesses include narrow landing margins; never equate62px displacement with a hard maximum gap or call a72px gap crystal-required.
- An8-emitter research layout using the existing staircase geometry passed3 jumps/0 dashes/0 deaths. Among651 fixed-route variants inserting one extra dash,387 reached the same landing with0 deaths, and none used fewer than3 successful jumps. This is a bounded witness study, not exhaustive reachability, uniqueness or human-balance proof.
- Temporary direct calls to the existing `Jump` motion were used only to estimate proposed air-jump trajectories. They do not implement crystal pickup, fresh-press input, consumption/respawn, Unity scene behavior or product delivery.

Source: `<project-root>/Assets/Phase/Runtime/PhaseController.cs`, `Laser.cs`, `PhaseDemoRooms.cs`; artifacts under `<user-documents>/Codex/2026-10-17/gamejam1-lasers/` and `phase-laser-level-research/`.
