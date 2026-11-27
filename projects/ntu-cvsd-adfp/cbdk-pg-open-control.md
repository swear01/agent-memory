---
title: CBDK ICV 的 PG 開路應核對 port compare 與萃取網表
scope: projects/ntu-cvsd-adfp
status: verified
updated: 2026-11-28
tags: [cbdk, icv, lvs, pg, sram]
---

# CBDK ICV 的 PG 開路應核對 port compare 與萃取網表

SCE-14 在 CBDK v2.5 `ICV_LVS.rs` 與 IC Validator Y-2026.03-SP1-1 用同一份 PG harness 比較原 GDS 與故障副本。原 GDS 的 24 顆 SRAM 都在 `SCE.net.gz` 接到兩個頂層 PG ports。

- 只切斷 VDD top label 到 PG 網的金屬路徑，保留 label 和 vias。原 deck 的 `text_open_rename` 計數維持 29。這 29 組是原生 SRAM signal，在 M1–M4 的同名標記未導通。
- 正式 `SCE.LVS_ERRORS` 在斷路副本新增 `schematic port net equated to layout non-port net`。萃取網表中，24 個 SRAM VDD pins 同時從頂層 `VDD` 變為 non-port net `2`。判定 PG 開路時應同時核對 comparator 和萃取網表。用實際金屬斷路對照證明 deck 對目標故障有反應。
- 以 PG harness 所得的通過範圍止於指定 PG 端點。全晶片比較仍需同次 production GDS、含 PG 網表和適用的實體 views。

證據在 SCE 報告 `exp/area/safe-fp/cj12-narrow-deck.md`，提交 `47bb5382f81957a967fc5a51d58123e0f0f33ec5`。
