---
title: CVSD Lab3 synthesis and SDF simulation closeout
scope: projects/ntu-cvsd-adfp
status: submitted-and-file-verified; grading unverified
updated: 2026-10-14
---

# Lab3 completed on 2026-10-13

- This supersedes the earlier preparation-only snapshot. Actual ADFP execution used Design Compiler 2026.03 and VCS 2024.09-SP2; both RTL and SDF-backed gate simulation passed the original 327,936 test cases.
- The unchanged course ALU and official inputs were preserved. The two functional edits enable `write -format verilog -hierarchy -output ALU_GTECH.v` before `compile`, and `$sdf_annotate("ALU.sdf", my_alu)` in a separate gate testbench. Derived files normalize line endings; original materials remain byte-identical.
- GTECH_OR2 count was 17. SDF annotation completed with 0 errors and 0 warnings. Reported setup slack was +1.72 ns under the supplied 10 ns clock constraint. These are tutorial results, not post-layout or multi-corner signoff.
- Zero warnings applies specifically to SDF annotation. Supplied synthesis operating-condition and unconnected-port warnings were retained and documented; do not describe the complete flow as warning-free.
- The three-page PDF was submitted to COOL at 19:47 Asia/Taipei, before the 20:10 deadline. Downloaded submitted bytes matched the uploaded PDF. Submission receipt and file equality do not establish TA acceptance or a grade; do not resubmit merely because a later session or Git change occurs.
- Private course Git records were merged in PR #11 after a clean Codex review of the latest head. Source, PDF and selected evidence were archived; Drive file names, sizes and hashes were read back. Raw ADFP evidence-tar download did not complete, so no local raw-log readback is claimed.

## Reusable execution and teaching notes

- The actual stages were VCS RTL simulation, `dc_shell -f synthesize.tcl`, then VCS gate simulation using the supplied cell-library file list and the enabled SDF testbench. Enabling two lines is the source change; running these stages and inspecting outputs is the lab work.
- `synthesize.tcl` loads libraries and RTL, exports GTECH, sets clock/I/O/load/optimization constraints, checks and compiles the design, then exports timing/power/area reports, mapped Verilog, SDF, SDC and the design database. The script itself does not run functional simulation.
- The verification runner preserves existing results by requiring a new run directory, checks official hashes, rejects DC errors and zero GTECH count, requires explicit simulation PASS markers and completed zero-diagnostic SDF annotation. A tool exit code alone is insufficient.
- The user said they now understand the walkthrough. Resume explanations from the distinction between two enabled script lines and the three actual execution stages; keep examples concrete and separate GTECH, mapped netlist and SDF.
