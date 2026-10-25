---
title: cthulhu 與 valkyrie 共用 NFS home
scope: machines/cthulhu
machine: cthulhu
status: verified
updated: 2026-10-25
tags: [nfs, valkyrie, cleanup]
---

cthulhu 與 valkyrie 的使用者 home（`<remote-home>`）是同一個 NFS home，兩台看到的是同一份檔案；在其中一台刪檔，另一台也會消失。清理時只需處理一次，刪除後列表可能因 NFS cache 短暫仍顯示舊項目。

HAPI 新 session 的目錄清單常會在這裡堆積 `<task>-YYYYMMDD` 形式的工作資料夾（例如 2026-10-25 清掉的 `cvsd-hw1-*`）。清理用 `gio trash`（進 `~/.local/share/Trash`），要真正釋放空間需 `gio trash --empty`。
