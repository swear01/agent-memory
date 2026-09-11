---
title: DVLab fleet 的 Synopsys Verdi X-2025.06 與 VCS FSDB 整合
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-11
tags:
  - synopsys
  - verdi
  - fsdb
  - vcs
  - eda
---

# 已確認環境

- Verdi X-2025.06 for linux64（2025-05-31 build）安裝在實驗室 EDA 共用安裝根目錄 `<lab-eda-root>/verdi/verdi/2025.06`；同層的 `cur` 是指向該版本的 symlink。
- 該目錄位於實驗室 NAS 的 NFS 掛載，因此五台 DVLab Linux 主機（Mazu、Cthulhu、Athena、Valkyrie、Zeus）共用同一份 Verdi 安裝。
- 設定腳本是共用 CIC 目錄下的 `verdi.sh`（與 `vcs.sh`、`spyglass.sh` 同層）：它設定 `VERDI_HOME` 與 `NOVAS_HOME`、把 `bin`／`nLint/bin`／`platform/LINUX64/bin` 加入 PATH、把 `share/PLI/VCS/LINUX64`、`share/PLI/lib/LINUX64`、`etc/lib/libstdc++/LINUX64` 加入 `LD_LIBRARY_PATH`、設定 `MANPATH`，最後載入既有 `license.sh`，並 alias `debussy='verdi'`。
- 與 `vcs`、`spyglass` 不同，fleet 上沒有對應的 Verdi launcher：`/usr/local/bin/verdi` 不存在，使用者必須自行 source `verdi.sh`（此項未部署，見下）。

# 已驗證的端到端流程

- 在 Mazu 以一個同時 source `vcs.sh` 與 `verdi.sh` 的 Bash shell 完成完整流程，不需手動設定 `VERDI_HOME`：

```bash
source <lab-eda-root>/synopsys/CIC/vcs.sh
source <lab-eda-root>/synopsys/CIC/verdi.sh
vcs -full64 -sverilog -debug_access+all -kdb -o simv counter.v tb.v   # tb 內含 $fsdbDumpfile/$fsdbDumpvars
./simv                                                                # 產生 dump.fsdb
fsdbreport dump.fsdb -s "/tb/dut/q" -bt 0 -et 220 -o q.txt            # headless 讀值
```

- 編譯輸出可看到 VCS 自動連結 `<lab-eda-root>/verdi/verdi/2025.06/share/PLI/VCS/LINUX64/pli.a`，並印出 `Verdi KDB elaboration done and the database successfully generated`。
- 執行 simv 時 FSDB Dumper 印出 `Release Verdi_X-2025.06`，產生 `dump.fsdb`。
- `fsdbreport` 讀出的 4-bit counter 值由 `0000`、`0001`、`0010` … 依序遞增到 `1111` 後翻回 `0000`，與 RTL 行為一致。
- Verdi 預設 log 目錄是執行目錄下的 `verdiLog`（`fsdbreportLog` 同理）。
- `vcs` 輸出中會出現一行無害的 `sh: 1: Syntax error: Bad fd number`，不影響編譯、連結或 FSDB 產生；性質與 SpyGlass 的 dash／Bash 混用問題同類。

# 已確認的陷阱

- 只用 fleet 既有的 `/usr/local/bin/vcs` launcher（它只 source `vcs.sh`）編譯含 FSDB dump 的 testbench 會失敗：VCS 回報 `Undefined System Task call to '$fsdbDumpfile'`、`Undefined System Task call to '$fsdbDumpvars'`，編譯結束碼 255。
- 根因不是 VCS：`vcs.sh` 內把 `VERDI_PLI` 指向 `<lab-eda-root>/design_compiler/verdi/cur/share/PLI/VCS/LINUX64`，而該路徑在此安裝中不存在，所以 `Verdi PLI linked.` 這行永遠不會出現。VCS 只有在 `VERDI_HOME` 被設定時才會找到 Verdi PLI，因此實際上是靠 `verdi.sh` 補上。
- 已驗證可行的修法：在同一個 shell 也 source `verdi.sh`（本次流程即採此法）。另一個等價修法是修正 `vcs.sh` 的 `VERDI_PLI` 路徑；未部署，因為該檔屬於共用安裝、非本作業帳號可寫。

# GUI 與 headless 邊界

- Verdi 是 GUI 工具。Mazu 目前沒有執行中的 X server，也沒有安裝 `Xvfb`、`xvfb-run`、`x11vnc` 或 `vncserver`；`/tmp/.X11-unix` 下雖然留有 `X1024`、`X1025` socket，但那是 gdm-greeter 留下的死 socket，`xdpyinfo` 連不上。遠端使用同樣需要 VNC 或 X11 forwarding，與 SpyGlass 的條件一致。
- 無 DISPLAY 時 `verdi -batch -ssf dump.fsdb -play <tcl>` 仍然以 `*ERROR* Failed to create DISPLAY.` 結束（退出碼 1），所以 `-batch` 在此環境不能用來做純文字驗證。
- 無 DISPLAY 的批次驗證請改用 `fsdbreport`（已驗證可讀 FSDB 並輸出訊號值）。Xvfb 方案未部署、也未驗證。

# 維運規則

- 學生要用 FSDB／Verdi 時必須同時載入 VCS 與 Verdi 環境；不要假設 `/usr/local/bin/vcs` 已經帶上 Verdi PLI。
- 排查順序：`command -v verdi`、`echo "$VERDI_HOME"`、`echo "$LM_LICENSE_FILE"`，再用最小 FSDB smoke test 確認 PLI 與 FSDB 產生；不要只看 `verdi -version` 是否印得出來。
- 若要補上 `/usr/local/bin/verdi` launcher 與修掉 `vcs.sh` 的 `VERDI_PLI` 路徑，需要具備共用安裝寫入權限的帳號執行，屬 fleet 層級的變更。
- 完整操作紀錄與現況仍以 DVLab MIS runbook／HackMD 為準；shared memory 只保存可重用的環境事實與整合決策。
