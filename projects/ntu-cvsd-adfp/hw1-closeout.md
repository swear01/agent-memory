---
title: CVSD HW1 作業封裝、模擬及 formal 驗收
scope: projects/ntu-cvsd-adfp
status: verified
updated: 2026-10-01
---

- **PR 有修改，或 OCR 把 v1 看成 vl。** 先核官方 `115-1_HW1.pdf`。第一版是 `cvsd_hw1_v1.tar.gz`，包內只有必要目錄、RTL `alu.v` 和 `rtl.f`。要在 ADFP home 放置並在 COOL 上傳，兩邊檔名、樹與逐檔 hash 一致即可；獨立打包 metadata 可以不同。以 COOL 回執下載核對上傳檔。PR 只改 runner/oracle 時先確認包內兩檔相同，勿覆蓋既有提交。
- **官方 status 或 exit 0 掩蓋真失敗。** VCS `$fatal` 可能 exit 0，還須 PASS marker 且無錯誤／reset diagnostics；`tee` wrapper 用 pipefail。Lint 的零錯誤應驗完整數值，避免 10 被當 0。Oracle 校驗用明確例外，避免 Python `-O` 移除 assert。驗 Atan2 quantized MSE=0 並逐筆核對，包含四象限、軸線、最小負數、half-LSB；nearest tie 朝正無限大。非零 MSE 的官方差值有溢位風險。
- **新增 spec 外硬體支援或擴大 formal 結論。** 依規格 rotate b=0..16、Gaussian a∈[-1,1]、矩陣八個有效輸入完才發下一指令。測官方 I0–I10。Formal 執行 `bash formal/run.sh`，以 XML 核對 proved、covered、non-vacuous 和 reset／合法 opcode assumptions；不把控制／狀態證明外推成完整數學 datapath proof。跑中途 reset 和隔離 mutation 驗 harness 能拒錯。
- **ADFP launcher 只驗 VPN stub，或只看到 review SUCCESS。** 讀原測試／實際會話紀錄再判斷 VPN、CDP、上傳或 review。gate off／skipped／增量零不是最新 head 的完整 passing review。源碼與作業證據保留私人 `swear01/CVSD2026`；先讀已繳交 receipt，避免把本機舊工作樹當目前版本。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/ntu-cvsd-adfp/hw1-closeout.md)
