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
  - nSchema
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

# Schematic View（nSchema）

- Verdi 的 schematic view 就是 nSchema。開啟前必須先載入設計資料，兩種都實測可行：KDB（`-dbdir simv.daidir`，需以 `-kdb` 編譯）或原始碼（`-f filelist.f`）；只載入 FSDB 不足以產生 schematic。
- GUI 選單路徑（取自 Verdi X-2025.06 Command Reference）：nTrace 視窗的 `Tools -> New Schematic from Source -> New Schematic`（依 design browser 目前的 active scope 產生）；同層另有 `Browser Window`、`Flattened Window`、`Hierarchical Flattened View`、`Fan-in`、`Fan-out`、`Driver`、`Load`、`Connectivity`、`Clock Tree`、`Reset Tree`、`ECO Window`。nSchema 視窗本身則有 `Tools -> New Schematic -> From Trace Results`（bind key `O`）、`Active Fan-in`、`Bus Contention`、`Editable Window for Selected/All`。
- 等價的 Verdi Tcl 指令（已實測，可用 `verdi -play <file>.cmd` 執行）：

```tcl
set s [schCreateWindow -delim . -scope tb.dut]      ;# 依 scope 產生
schFit -win $s
set d [schCreateWindow -delim . -driver -signal {tb.dut.q}]   ;# 依 trace driver 結果
schCapture -win $s -file out.png -region 0 0 5000 5000        ;# 匯出 PNG，回傳 1 表成功
```

- `schCapture` 會把目前 nSchema 視圖輸出成 PNG，適合放進實驗報告；不需要互動截圖。

# GUI 與 headless 邊界

- Verdi 是 GUI 工具。Mazu 目前沒有執行中的 X server，系統也沒有安裝 `Xvfb`／`xvfb-run`／`x11vnc`／`vncserver`；`/tmp/.X11-unix` 下雖然留有 `X1024`、`X1025` socket，但那是 gdm-greeter 留下的死 socket，`xdpyinfo` 連不上。日常使用仍需 VNC 或 X11 forwarding，與 SpyGlass 的條件一致。
- 無 DISPLAY 時 `verdi -batch -ssf dump.fsdb -play <tcl>` 仍然以 `*ERROR* Failed to create DISPLAY.` 結束（退出碼 1），所以 `-batch` 在此環境不能當作 headless 模式。
- 純文字讀值請用 `fsdbreport`（已驗證）。
- Verdi 安裝內自帶 `Xvfb`：`<lab-eda-root>/verdi/verdi/2025.06/bin/Xvfb`（symlink 到 `.wrapper`，實際 binary 在 `platform/LINUX64/bin/Xvfb`）。以 `.../bin/Xvfb :99 -screen 0 1920x1080x24 -nolisten tcp -fp /usr/share/fonts/X11/misc` 啟動後，`DISPLAY=:99 verdi …` 可完整跑起 nTrace／nWave／nSchema，不需要 root 安裝任何套件。
- `-fp /usr/share/fonts/X11/misc` 是必要的：這個 Xvfb 的內建 font path 是舊的 `/usr/X11R6/...`，Ubuntu 26.04 不存在，會以 `Fatal server error: could not open default font 'fixed'` 結束。`-fp` 只接受單一路徑，用逗號串多個路徑會被當成一個不存在的目錄。
- 訊息 `Couldn't open RGB_DB '/usr/X11R6/lib/X11/rgb'` 與 `error opening security policy file` 都是無害警告；設 `RGB_DB`／`SPS_RGB_PATH` 環境變數對這個 binary 無效（路徑是編譯期寫死的）。
- `DISPLAY=:99 xwd -root -silent -out shot.xwd` 加 `ffmpeg -i shot.xwd shot.png` 可在無桌面環境擷取畫面；`xdotool`／`wmctrl` 未安裝，無法做點擊自動化。
- 用 `pkill -f verdi` 清 Verdi 會連 Xvfb 一起殺掉（路徑含 `verdi`）。要清 GUI 請用 `pkill -x Novas`（`verdi` 是 wrapper，真正 GUI 程序名是 `Novas`）。

# 維運規則

- 學生要用 FSDB／Verdi 時必須同時載入 VCS 與 Verdi 環境；不要假設 `/usr/local/bin/vcs` 已經帶上 Verdi PLI。
- 要看 schematic 就是 nSchema；先確定設計已載入（`-dbdir` 或 `-f`），再用 `Tools -> New Schematic from Source`。
- 排查順序：`command -v verdi`、`echo "$VERDI_HOME"`、`echo "$LM_LICENSE_FILE"`，再用最小 FSDB smoke test 確認 PLI 與 FSDB 產生；不要只看 `verdi -version` 是否印得出來。
- 若要補上 `/usr/local/bin/verdi` launcher 與修掉 `vcs.sh` 的 `VERDI_PLI` 路徑，需要具備共用安裝寫入權限的帳號執行，屬 fleet 層級的變更。
- 完整操作紀錄與現況仍以 DVLab MIS Google Drive runbook 為準；shared memory 只保存可重用的環境事實與整合決策。
