---
title: Check both clock-gate edges in SDF post simulation
scope: project/cvsd-final-sce
status: active
updated: 2026-11-28
---

SCE-133 exposed a model gap on the original CBDK negative-timing Verilog. Native Innovus setup, hold, and all SRAM pulse checks passed at 1.780 ns. VCS still reported 1,138 TLATX1 setup/hold violations before any output. All of their clock cones traced to 11 TLATNCA gated-clock instances.

For the first failing latch, the saved RC/SDF expected a 313.3 ps CK-to-ECK fall delay from the TLATNCAX20 gate. The vendor simulation model defines the rising CK path only. Replaying the same VCS executable at 100 fs showed CK and ECK falling in the same time slot while enable and its latched value were high. The rising path took the annotated 365 ps. Omitting the falling arc from an SDF path walk predicted latch G to within 0.5 ps of the waveform. Its real D fell 151 ps before G, inside the annotated 159/−150 ps setup/hold window. The latch notifier changed and Q became X in that time slot. The later system stall was not traced back through every intervening state.

When propagated latch STA reports zero or time borrowing and post simulation first fails at a latch, trace its G driver back to the gate. Compare native CK/ECK edge arcs, the exact SDF instance section, and the original Verilog specify path by edge. An aggregate IOPATH annotation PASS does not establish that the falling edge traverses a valid specify path. Use the same compiled executable to observe gate CK/E/ECK and latch G/D/dG/dD/NOTIFIER/Q around the first event. Keep timing checks and notifiers enabled. Establish the cell-model edge gap before changing SRAM clocks or retiming latch data.

Evidence: SCE-133 in the MAIN ledger, including the saved-DB native path, SDF, full PAT0, and the two short VCS replays.
