---
title: Mazu 管理操作使用 swear02
scope: machines/mazu
status: active
updated: 2026-10-19
---

使用者指定 Mazu 管理操作使用 `swear02`。實測 SSH 登入成功，`sudo -n -l` 顯示 `NOPASSWD: ALL`；`swear01` 無 sudo 權限。

讀核心日誌需透過 `sudo -n journalctl -k -b --no-pager`。`swear02` 直接執行 journalctl 仍可能提示無系統日誌權限，不應因此判定沒有紀錄或無法取得管理權限。

本機 SSH alias `su_mazu` 即 `swear02`（`mazu` alias 為 `swear01`）。`swear01` 執行 sudo 時，不論有無 TTY 都直接回 `sudo: I'm sorry swear01. I'm afraid I can't do that`，不會要求密碼；看到此訊息應改用 `ssh su_mazu 'sudo ...'`，而不是排查密碼或 TTY。2026-10-19 以此方式成功部署 spec2rtl CI controller release，見 [[projects/spec2rtl/controller-release-staging]]。
