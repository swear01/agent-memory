---
title: GUPS Windows X11 / Innovus GUI 學生指南（ChatGPT Sites）
scope: projects/ntu-gups
project: ntu-gups
status: active
updated: 2026-11-04
---

- [GUPS 學生指南](https://windows-x11-eda-guide.swear01.chatgpt.site) 是公開單頁 `dist/index.html`，附 `dist/start_innovus_gui.ps1`。Windows 驗證使用 VcXsrv 和 MIT-MAGIC-COOKIE reverse tunnel，腳本的 `RemoteCommand` 預設 `xterm`。發布另需使用者核准，見 [[exec-sites-deploy-from-claude]]。
- ICD001 登入 shell 為 bash，有 `xterm`、`xwininfo`、`xdpyinfo`、`/usr/bin/Xvfb`。因沒有 `xclock`／`xeyes`，用 `xterm` 測 X11。啟動用 `source /cad/cadence/CIC/innovus.sh` → `cd ~/innovus_u18_tutorial/run` → `innovus -stylus`。Xvfb `:93` 下曾驗證啟動，沒視窗先試 `gui_show`。
- 教學留登入、測視窗、開 Innovus、備案及常見問題。依使用者要求去掉狀態標籤、證據和 PuTTY。相關 [[eda-environment]]、[[eda-course-accounts]]。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/ntu-gups/windows-x11-eda-guide-site.md)
