---
title: Inari Glances 使用 managed Python，避開未安裝開發工具的系統 stub
scope: machines/inari Glances monitoring
status: deployed and API verified 2026-10-28; reboot untested
updated: 2026-10-28
---

- 此機 `/usr/bin/python3` 是需要 Xcode／Command Line Tools 的 stub；未安裝開發工具時不能拿它建立 Glances venv。系統 stub 保持原狀。
- 本次安裝官方 uv `0.13.6` 並核對 release digest，root 使用的 binary 為 `/opt/dvlab-tools/uv`；managed CPython `3.12.16` 位於 `/opt/dvlab-python/cpython-3.12.16-macos-aarch64-none`。
- Glances `4.5.8` 安裝在 `/opt/glances-venv`，由 root-owned LaunchDaemon `tw.dvlab.glances` 常駐。installer 用 sudo 執行時，PATH 必須能解析 managed Python；重跑前先確認，不要回到系統 stub。
- 已從 inari 與 collector 主機確認 Glances load／mem／fs API HTTP 200。實際 APFS Data filesystem 有效且未回傳 devfs；live collector 與內建 browser 都顯示 online。
- 未重開機測試。安裝腳本最後只列印 API 狀態，不能把 shell exit 0 當作 API 成功；部署後需另讀回 HTTP 狀態及 collector 資料。
