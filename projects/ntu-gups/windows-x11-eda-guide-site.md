---
title: GUPS Windows X11 / Innovus GUI 學生指南（ChatGPT Sites）
scope: projects/ntu-gups
project: ntu-gups
status: active
updated: 2026-11-04
---

# GUPS Windows X11 / Innovus GUI 學生指南

- 網址 `https://windows-x11-eda-guide.swear01.chatgpt.site`，project `appgprj_6aea13df7d588191af05fc758f05d075`，public。單頁 `dist/index.html` + 下載檔 `dist/start_innovus_gui.ps1`（VcXsrv + 自建 MIT-MAGIC-COOKIE 的 reverse tunnel 腳本，使用者已在真實 Windows 測過可用）。
- 2026-11-04：v3 是另一個 session 改的繁中版（仍叫學生跑 `xclock`）。Claude 重寫成白話繁中版並存為 v4（`appgver_0413d5c77bb88191846df8b9bbdd2544`，source commit `bdf7e25`），腳本預設 `RemoteCommand` 由 `xclock` 改為 `xterm`；發布需使用者核准，見 [[exec-sites-deploy-from-claude]]。
- 工作站事實（以 ICD001 實測）：**沒有 `xclock`／`xeyes`**，有 `xterm`、`xwininfo`、`xdpyinfo`、`/usr/bin/Xvfb`；登入 shell 為 bash。測 X11 用 `xterm`。
- 開 Innovus GUI：`source /cad/cadence/CIC/innovus.sh` → `cd ~/innovus_u18_tutorial/run` → `innovus -stylus`；在 Xvfb `:93` 下實測 GUI 啟動、無 `IMPSYT-1507` no-window 警告。沒視窗時 `gui_show`。
- 使用者要求網頁「說人話」：拿掉「已實測／僅核對文件」等狀態標籤、證據段落與 PuTTY 段，只留登入 → 測視窗 → 開 Innovus → 備案 → 常見問題。

相關：[[eda-environment]]、[[eda-course-accounts]]
