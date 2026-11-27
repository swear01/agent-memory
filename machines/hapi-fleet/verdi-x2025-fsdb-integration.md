---
title: DVLab fleet 的 Synopsys Verdi X-2025.06 與 VCS FSDB 整合
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-16
tags:
  - synopsys
  - verdi
  - nSchema
  - fsdb
  - vcs
  - eda
---

# Verdi X-2025.06：VCS FSDB 與 nSchema

- 於同一新 Bash 分別 source `/apps/eda/synopsys/vcs.sh <version>` 和 `/apps/eda/synopsys/verdi.sh 2025.06`，再執行 `vcs -full64 -sverilog -debug_access+all -kdb -o simv counter.v tb.v`，TB 需有 `$fsdbDumpfile`／`$fsdbDumpvars`。只看工具找到或 launcher 自選 VCS 不能印證已載 Verdi PLI。校驗 `VERDI_HOME`、VCS link 到 `share/PLI/VCS/LINUX64/pli.a`、KDB 生成、`simv` Dumper 版本與實際 `dump.fsdb`。舊 VCS-only launcher 曾用不存在的 `VERDI_PLI` 路徑，造成 `Undefined System Task call to '$fsdbDumpfile'`／`'$fsdbDumpvars'` 和 exit 255。修正共用路徑當時未部署，學生走明確雙選版。
- 純文字可執行 `fsdbreport dump.fsdb -s "/tb/dut/q" -bt 0 -et 220 -o q.txt`，驗代表訊號與 RTL 是否一致。Verdi 預設 log 為執行目錄內 `verdiLog`。特定舊版的 `Bad fd number` 曾不影響輸出，仍核對實際 FSDB。
- nSchema 要先載設計 `-dbdir simv.daidir`（編譯時需 `-kdb`）或 `-f filelist.f`，只載 FSDB 可看波形但不足以產生 schematic。nTrace 的 `Tools → New Schematic from Source → New Schematic` 對 active scope；需要擷取 PNG 可在 `verdi -play <file>.cmd` 執行：

```tcl
set s [schCreateWindow -delim . -scope tb.dut]
schFit -win $s
schCapture -win $s -file out.png -region 0 0 5000 5000
```

- `verdi -batch` 仍可能報 `Failed to create DISPLAY`。先以 `xdpyinfo` 查實際 display，`/tmp/.X11-unix` 可能有死 socket。可用既有 X11/VNC 或 Verdi 2025.06 內附 `/apps/cad/verdi/verdi/2025.06/bin/Xvfb`，例如 `:99 -screen 0 1920x1080x24 -nolisten tcp -fp /usr/share/fonts/X11/misc`。舊 binary 預設 font path 不存在時報找不到 `fixed`，`-fp` 只給一個存在目錄。設 `DISPLAY=:99` 後才跑 GUI；要整屏畫面可用 `xwd` 加 ffmpeg。清自己的 GUI 時用確認過的 PID，廣泛 `pkill -f verdi` 會連安裝路徑有 Verdi 的 Xvfb 一起殺掉，GUI process 通常為 `Novas`。
- 2026.03 的 FSDB 自動 PLI／PrimeTime 讀回條件見 [EDA 安裝盤點](eda-install-inventory.md)，不可將兩版的 PLI 行為互推。

[舊 launcher、各 GUI 路徑與端到端模擬收據](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/machines/hapi-fleet/verdi-x2025-fsdb-integration.md)
