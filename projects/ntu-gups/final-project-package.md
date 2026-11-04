---
title: GUPS EDA Final Project 教材包（innovus_u18_tutorial）位置與 2026-11-04 改版
scope: projects/ntu-gups
project: ntu-gups
status: active
updated: 2026-11-04
---

# GUPS EDA Final Project 教材包

## 正本在哪

- 正本是整包資料夾：swop（Windows）`<swop-home>/Documents/gups_final/innovus_u18_tutorial/`。`~/Downloads/homework_gui.html` 只是使用者方便看的副本，不是正本；改版要回寫整包。
- 工作站 `/cad/tutorial/innovus_u18_tutorial/`（學生 `cp -r` 與 `scp homework/` 的來源）與 Google Drive「GUPS助教/innovus_u18_tutorial」都還是 10-26 舊版，部署到工作站是另一步。
- 學生入口 `gui/homework_gui.html`（英文 EMI，0 CJK），圖在 `gui/images/*.webp`；`gui/gui_flow.html` 只是 03–08 圖文參考（不含評分）；根目錄 `gui_flow.html` 是同內容、路徑加 `gui/` 前綴的副本。`homework/answers.html` 以 img/script 載入檢查繳交檔是否存在（`.tcl|.txt|.html` 的 error 事件要 preventDefault）。

## 2026-11-04 改版決定（TA 提案，待老師確認）

- 老師要總分 120：基本 100 = 共同 00–02 25 + Part A CLI 45 + Part B GUI flow 30；Exploration 加分 20（X1–X5 各 4，X6 自選可替換一項）。
- Part B 改成一次走完的 baseline（B01 Verisium 波形、B03–B08 表單），原本「variant→restore」移到 Exploration，各自從 `gui_0X` checkpoint 開新 session、不覆寫 `gui_*`。B02 Genus GUI 暫缺：`genus` 有 DISPLAY 會開 GUI、有 `gui_schematic`，但選單路徑未實測。
- 回寫前先比對 swop 與快照 MD5，備份 `gups_final/backup_20261104_before_regrade.zip`。

## 工作站實測事實（Innovus/Genus 25.10，Xcelium 25.03）

- `get_db rows`（23 列）、`get_db designs .core_bbox`、`keep` helper、`cts_max_fanout`（預設 100，07_cts.tcl 用 16）、`report_clock_trees/report_skew_groups -summary` 都可用。
- hold 報告是 `reports/07_timing/hold_hold.summary.gz`（`time_design -report_prefix hold`），09 同理。
- Top layer：`route_design_top_routing_layer` 已過時（NRIF-91），用 `design_top_routing_layer`。本 UART signal 只用 metal1–4，限 metal4 幾乎無差；設 3 會因 CTS 已有 metal4 線 `NRDB-954` 中止。
- 快捷鍵：ruler `K`、fit `F`、zoom `Z`（來源 `INNOVUS251/etc/innovus/uiView.tcl`）。
- 驗證方式：`ssh innovus-server`（root）後 `su - gupstest`，在 scratch copy 跑 `innovus -stylus -batch`，結束刪除。
