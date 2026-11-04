---
title: GUPS EDA Final Project 教材包（innovus_u18_tutorial）位置與 2026-11-04 改版
scope: projects/ntu-gups
project: ntu-gups
status: active
updated: 2026-11-04
---

# GUPS EDA Final Project 教材包

## 正本在哪

- 正本是整包資料夾：swop（Windows）`<swop-home>/Documents/gups_final/innovus_u18_tutorial/`。Downloads 裡的 html 只是使用者方便看的副本，不是正本；改版要回寫整包，回寫後用 MD5 逐檔比對。
- 工作站 `/cad/tutorial/innovus_u18_tutorial/`（學生 `cp -r` 與 `scp homework/` 的來源）2026-11-04 已部署新版：`git archive HEAD` 排除 `ta_notes/`，保留 `run/`，root:PDK_approved、目錄 750、檔案 640，43 檔 MD5 與本地一致，gupstest `cp -r` 實測可讀。部署前備份在工作站 root 家目錄 `backups/innovus_u18_tutorial_20261026.tgz`。Google Drive「GUPS助教/innovus_u18_tutorial」同日也同步成新版（舊版備份 `innovus_u18_tutorial_backup_20261026.zip`，TA 筆記放 `innovus_u18_answer_key/ta_notes/`）。2026-11-04 已從 NTU 信箱寄新版 zip 給王信博老師請他確認，老師同意後，2026-11-04 已發佈作業並在 COOL 發公告 "Final Project released"（英文，請同學有問題找助教）。NTU COOL（GUPS3006，使用者是 TA）的「EDA Final Project」作業（120 分、12/25 截止）同日已附整包 `innovus_u18_tutorial.zip`（放隱藏的 Uploaded Media）。作業說明使用者要求「能少放就少放」，只放必要訊息：解壓後用瀏覽器開 index.html、怎麼交 zip，加附檔連結；不要介紹作業、不要抄教學頁內容、不要重寫 deadline（COOL 的 Due 欄位已顯示）；舊的單檔 index.html 還留在 Files。內建瀏覽器的頁面 fetch 不到 localhost，上傳檔案改用 Canvas files API：頁面內取 upload slot → shell `curl -F` 上傳 → 頁面內呼叫 `create_success`。比對 MD5 清單時兩邊都要 `LC_ALL=C sort`，否則 Linux 與 macOS 排序不同會誤判。
- 備份：`gups_final/backup_20261104_before_regrade.zip`、`backup_20261104_before_singlepage.zip`。

## 版面（2026-11-04 定版：教學頁 + 答案卷兩個檔）

- `index.html` 是唯一教學頁（英文 EMI，0 CJK）：評分→00 Setup（環境、X11、MobaXterm 連線）→Part A CLI（01 模擬、02 合成、03–10）→Part B GUI（B01–B08）→Exploration→Submit。不要再拆 `gui/`、`homework_gui.html`、`windows_gui.html`；不要「Shared steps」分組（01/02 是有評分練習的 CLI 步驟）。
- 所有表格與問答在 `homework/answers.html`（沿用 10-26 原版答案卷：Name/ID、Save answers 用 File System Access API 存回同檔、Files 區直接顯示同資料夾的截圖並標 Found/Missing，`.tcl/.txt` 用 `<script>` 探測）。教學頁只留 Hand in 勾選與 `answers.html: …` 指引，不放輸入框、不放上傳預覽。使用者曾明確要求「照原本的做法」，不要把答案併進教學頁。
- 答案卷數字題拆成獨立小格並用 `pattern` 做格式檢查（整數、小數、8 bits、hex byte、cell 名、LSB/MSB），只有簡答與申論題用大 textarea；使用者明確要求這樣比較好批改。
- 教學頁 Submit 用 `<pre class="tree">` 畫出繳交資料夾樹（`homework/` 以下每個檔案加說明，`.tree-note` 不能用 `.note`，會撞到既有提示框樣式）。新增繳交檔要同步改答案卷 Files 區、Submit 樹、Hand in 張數。
- Exploration 截圖不檢查 Found/Missing（每人做的不同），X6 可交任意多個自選設定。
- 圖在 `images/*.webp`；TA 專用 `ta_notes/`（`answer_key.md` 為實跑參考答案）。

## 配分（TA 提案，待老師確認）

- 總分 120：基本 100 = Part A CLI 65（01 練習 4、Ex1 6、02 4、Ex2 8、evidence 7、DRC 4、Ex3–5 各 7、Ex6 11）+ Part B GUI 35；00 Setup 不計分；Exploration 加分 20（X1–X5 為建議題，X6 自選設定不限數量，每項 4 分，上限 20）。使用者認可 Part B 份量大（越後面越難拿分）。
- Part B：B01 5、B02 4、B03 4、B04 4、B05 5、B06 4、B07 4、B08 5。
- X5 改為從 `gui_06_place` 先設 `design_top_routing_layer 3` 再做 CTS/route（從 `gui_07_cts` 設 3 會 `NRDB-954`，設 2 會 SEGV）。

## 工作站實測事實（Innovus/Genus 25.10，Xcelium 25.03，Verisium Debug 25.03）

- `get_db rows`（23 列）、`get_db designs .core_bbox`（`{15.5 15.12 141.36 131.04}`）、`cts_max_fanout`、`report_clock_trees/report_skew_groups -summary` 可用；hold 報告是 `hold_hold.summary.gz`。
- `route_design_top_routing_layer` 已過時（NRIF-91），用 `design_top_routing_layer`。
- Verisium：`cd sim/rx && verisium_debug -lwd xcelium.d -db ida.db &`，Hierarchy 分頁 tb → i_uart_rx 加訊號。參考圖要裁掉底部 value bar，否則會露出第 3 個 byte 答案。
- Genus GUI：有 DISPLAY 時 `genus` 直接開視窗。`syn.tcl` 最後一行改成 `if {![info exists KEEP_GUI]} { exit }`，學生 `set KEEP_GUI 1; source ../scripts/syn/syn.tcl` 後 GUI 不會關。Schematic：Tcl `gui_sv_load [get_db designs]` 實測可開，`gui_snapshot_sv -png -overwrite f` 存圖（實際輸出是 GIF，611×451）。選單路徑「Design Browser 右鍵 Hier Cell → Schematic View (Module) → In Main」只來自 25.1 GUI guide（`GENUS251/doc/genus_gui/design_browser_topics_re_Design_Browser_Component_Hier_Cell.html`），沒點到。
- 在 `genus -files x.tcl` 裡用 `vwait` 會卡住不進事件迴圈；用阻塞的 `after N` 即可。
- 驗證方式：`ssh innovus-server`（root）後 `su - gupstest`，在 scratch copy 跑，結束刪除。

## swop GUI 截圖的坑

- 遠端 X11（MobaXterm）視窗：用 Windows `SetCursorPos`/`mouse_event` 點擊座標要以截圖原始像素 ×1（視窗 rect 為實體像素），y 約偏一列；Qt 右鍵選單不會出現在 `CopyFromScreen` 截圖裡。能用工具自己的 snapshot 指令就別用滑鼠驅動。另見 [[interactive-desktop-screenshot]]。
- 已移除 VcXsrv 與 PuTTY 路線，只教 MobaXterm；`start_innovus_gui.ps1` 已刪。當時實測：`start_innovus_gui.ps1`（VcXsrv + SSH reverse tunnel）實測 `-Probe` 可用，但遠端 display 固定 178：第二個人同時連會 `remote port forwarding failed for listen port 6178`，全班只能一人用；且預設直接開 `innovus -stylus`（頁面卻寫會開 xterm），無金鑰時要輸入 4 次密碼。教學主線用 MobaXterm。
- 全螢幕截圖會拍到使用者私人視窗（Gmail 等），只截目標視窗 rect，拍到就刪。
