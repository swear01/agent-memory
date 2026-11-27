---
title: CVSD Lab 的 SDF、PrimeTime、Innovus 與共用 ADFP 陷阱
scope: projects/ntu-cvsd-adfp
status: verified
updated: 2026-11-28
---

- **忽略規則刪到 Lab1／Lab2 的必要輸入，或歸檔時覆蓋已交 PDF。** Git 留英文頂層 Lab 目錄和重跑必需的 source、bench、filelist、reference；老師提供的 `Lab2_alu_s.v`／`Lab2_alu_s.sdf` 與 Lab1 標準答案 `program_out.txt` 要保留，製程 library 只保留引用，不存大 FSDB/VCD、工具 binary 或 cache。Drive 各 Lab 分 `materials/`、`submission/`、`evidence/` 並用 SHA256SUMS 核原始 PDF、回條與必要證據。舊 `verify_adfp.sh` 依賴指定 home 輸入與固定日期目錄，先讀 README；歸檔不等於重跑、重交或評分。
- **Lab3 只有合成成功，卻說 gate sim 通過。** 分別跑 VCS RTL、`dc_shell -f synthesize.tcl`、VCS gate；`write -format verilog -hierarchy -output ALU_GTECH.v` 在 compile 前，SDF 由獨立 gate bench 的 `$sdf_annotate("ALU.sdf", my_alu)` 啟用。新 run 目錄保留官方 source hash、非零 GTECH count、DC 無 errors、明確的模擬 PASS 與 SDF annotation counts。零 annotation warnings 只限該階段，供應的 DC warnings 保留。
- **Lab4 沒看到 hold violation。** PrimeTime `report_timing` 預設 max/setup，hold 用 `report_constraint -all_violators` 或 coverage。最小週期用候選與鄰近較短週期兩次驗證；uncertainty 設零只是課堂控制，實際違反路徑要驗 hold 修復。課程 tcl 開頭無 design 時可有 `remove_design -all`／SEL-005，需從後續成功讀入及報告辨識。
- **Innovus `IMPFP-7235`／`IMPFP-7237` 互換。** Lab5 的 placement pitch x 0.46、y 0.41，不是 row height。用 `get_snap_grid_info -type placement` 讀 `-pitch_x`／`-pitch_y`／`-origin`，保留 `floorPlan -r` 的 core，將 die 上角調到 placement grid，`floorPlan -b die io core` 一次設好。Site 取 `top.fPlan.rows.site.size_x`，不是 head.sites 的第一個 corner。用 `dbGet -p2 top.insts.cell.name RF2SH64x16` 找 SRAM，25.1 用 `dbSet $inst.pStatus fixed`。需圖時 gups 的課程 Innovus 可 `xvfb-run -a innovus -files <tcl>` + `gui_dump_picture`；DVLab 的舊實測缺 64-bit libXp.so.6，i386 套件不能補。
- **多工作共用 ADFP，終端突然收到亂碼或停住。** 鍵盤送到同一 Guacamole 的前景視窗。先用既有 owner 的共享鎖、確認 target terminal 並通知其他同時作業者。取消持鎖指令可能留下無 PID 的鎖，先找 owner 核對或回收，勿把別人的終端當可關閉。新 terminal 為 tcsh，需 `cb` 再進 bash；下載失敗時先分段讀回必要輸出。COOL 已提交要讀回 receipt 和下載同一份檔案，是否需要繳交以本次抽查與規格為準。

[Lab3 原始驗證與繳交](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/ntu-cvsd-adfp/lab3-closeout.md) · [Lab4](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/ntu-cvsd-adfp/lab4-closeout.md) · [Lab5](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/ntu-cvsd-adfp/lab5-closeout.md)

[Lab1／Lab2 來源與繳交驗證](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/ntu-cvsd-adfp/lab-archive.md)
