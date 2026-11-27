---
title: GUPS EDA Final Project 教材包與工具陷阱
scope: projects/ntu-gups
project: ntu-gups
status: active
updated: 2026-11-04
---

# GUPS EDA Final Project 教材包

## 正本在哪
- 正本是整包資料夾：swop（Windows）`<swop-home>/Documents/gups_final/innovus_u18_tutorial/`。Downloads 裡的 html 只是使用者方便看的副本，不是正本；改版要回寫整包，回寫後用 MD5 逐檔比對。
- 工作站正本對應 `/cad/tutorial/innovus_u18_tutorial/`。打包 `git archive HEAD` 時排除 `ta_notes/`、保留 `run/`，root:PDK_approved、目錄 750、文件 640。跨 Linux/macOS 比對 MD5 前兩端都 `LC_ALL=C sort`，再用 gupstest 核 `cp -r`。Drive 的 GUPS助教 教材包與 `innovus_u18_answer_key/ta_notes/` 分開。
- COOL 上傳採 Canvas files API：頁面取 upload slot → shell `curl -F` → 頁面 `create_success`。內建瀏覽器 fetch localhost 曾失敗。作業說明僅放解壓開 `index.html`、zip 交件法及附檔連結，保留 Due 欄位為截止時間來源。正式配分查 COOL 和教材包。

- `index.html` 是唯一教學頁（英文 EMI，0 CJK）：評分→00 Setup（環境、X11、MobaXterm 連線）→Part A CLI（01 模擬、02 合成、03–10）→Part B GUI（B01–B08）→Exploration→Submit。不要再拆 `gui/`、`homework_gui.html`、`windows_gui.html`；不要「Shared steps」分組（01/02 是有評分練習的 CLI 步驟）。
- 所有表格與問答在 `homework/answers.html`（沿用 10-26 原版答案卷：Name/ID、Save answers 用 File System Access API 存回同檔、Files 區直接顯示同資料夾的截圖並標 Found/Missing，`.tcl/.txt` 用 `<script>` 探測）。教學頁只留 Hand in 勾選與 `answers.html: …` 指引，不放輸入框、不放上傳預覽。使用者曾明確要求「照原本的做法」，不要把答案併進教學頁。
- 答案卷數字題拆成獨立小格並用 `pattern` 做格式檢查（整數、小數、8 bits、hex byte、cell 名、LSB/MSB），只有簡答與申論題用大 textarea；使用者明確要求這樣比較好批改。
- 教學頁 Submit 用 `<pre class="tree">` 畫出繳交資料夾樹（`homework/` 以下每個檔案加說明，`.tree-note` 不能用 `.note`，會撞到既有提示框樣式）。新增繳交檔要同步改答案卷 Files 區、Submit 樹、Hand in 張數。
- Exploration 截圖不檢查 Found/Missing（每人做的不同），X6 可交任意多個自選設定。
- 圖在 `images/*.webp`；TA 專用 `ta_notes/`（`answer_key.md` 為實跑參考答案）。
- X5 改為從 `gui_06_place` 先設 `design_top_routing_layer 3` 再做 CTS/route（從 `gui_07_cts` 設 3 會 `NRDB-954`，設 2 會 SEGV）。

## 工作站實測事實（Innovus/Genus 25.10，Xcelium 25.03，Verisium Debug 25.03）

- `get_db rows`（23 列）、`get_db designs .core_bbox`（`{15.5 15.12 141.36 131.04}`）、`cts_max_fanout`、`report_clock_trees/report_skew_groups -summary` 可用；hold 報告是 `hold_hold.summary.gz`。
- `route_design_top_routing_layer` 已過時（NRIF-91），用 `design_top_routing_layer`。
- Verisium：`cd sim/rx && verisium_debug -lwd xcelium.d -db ida.db &`，Hierarchy 分頁 tb → i_uart_rx 加訊號。參考圖要裁掉底部 value bar，否則會露出第 3 個 byte 答案。
- Genus GUI：有 DISPLAY 時 `genus` 直接開視窗。`syn.tcl` 最後一行改成 `if {![info exists KEEP_GUI]} { exit }`，學生 `set KEEP_GUI 1; source ../scripts/syn/syn.tcl` 後 GUI 不會關。Schematic：Tcl `gui_sv_load [get_db designs]` 實測可開，`gui_snapshot_sv -png -overwrite f` 存圖（實際輸出是 GIF，611×451）。選單路徑「Design Browser 右鍵 Hier Cell → Schematic View (Module) → In Main」只來自 25.1 GUI guide（`GENUS251/doc/genus_gui/design_browser_topics_re_Design_Browser_Component_Hier_Cell.html`），沒點到。
- 在 `genus -files x.tcl` 裡用 `vwait` 會卡住不進事件迴圈；用阻塞的 `after N` 即可。
- 遠端 X11（MobaXterm）視窗：用 Windows `SetCursorPos`/`mouse_event` 點擊座標要以截圖原始像素 ×1（視窗 rect 為實體像素），y 約偏一列；Qt 右鍵選單不會出現在 `CopyFromScreen` 截圖裡。能用工具自己的 snapshot 指令就別用滑鼠驅動。另見 [[interactive-desktop-screenshot]]。
- 正式教材使用 MobaXterm。舊 VcXsrv/PuTTY 路徑的 fixed display 178 造成 `remote port forwarding failed for listen port 6178`，只能同時一人。網站獨立指南與課程包須各查實際發布。截圖只取目標視窗，避免拍入私人內容。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/ntu-gups/final-project-package.md)
