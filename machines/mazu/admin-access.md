---
title: Mazu 管理操作使用 swear02
scope: machines/mazu
status: active
updated: 2026-09-07
---

使用者指定 Mazu 管理操作使用 `swear02`。實測 SSH 登入成功，`sudo -n -l` 顯示 `NOPASSWD: ALL`；`swear01` 無 sudo 權限。

讀核心日誌需透過 `sudo -n journalctl -k -b --no-pager`。`swear02` 直接執行 journalctl 仍可能提示無系統日誌權限，不應因此判定沒有紀錄或無法取得管理權限。
