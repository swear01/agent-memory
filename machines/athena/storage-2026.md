---
title: Athena 本機磁碟與舊 Home 快取
scope: machines/athena
machine: athena
tags: [storage, backup, qmd, cache]
status: active
created: 2026-09-30
updated: 2026-09-30
---

# Athena 本機磁碟與舊 Home 快取

2026-09-30 重新開機後可由 Zeus 公網 SSH 與 Mazu LAN `192.168.1.202` 連線。唯一實體資料碟是 931.5 GiB NVMe；ext4 UUID `9953e63b-60be-4259-9264-ed927a610268` 同時掛在 `/`、`/mnt/md0`。現行 `/home` 是 Gaia NAS 的 NFS，不是第二份本機 Home。`df -B1 /` 已用 313,362,923,520 bytes、可用 618,907,582,464 bytes；未見外接硬碟。

本機 `/var/tmp/issue25-qmd-athena` 約 213.72 GB，`/var/tmp/issue25-v5-ops` 約 22.85 GB。前者 `generations/.../index.sqlite` 83,633,840,128 bytes 仍是 `publication-v5/active` 的目標，後者有 `issue25_downstream_thermal_guard.sh` 運行；不可把整棵 Issue 25 工作區當作可刪暫存。Docker 11 個 image 其中有本地 EDA 及研究用途；`reclaimable` 18.02 GB 不是安全刪除證據。

被 NFS 遮住的本機 `/mnt/md0/home` 刪除前為 10,740,592,640 bytes，幾乎全在 `piohuang/.cache/huggingface` 的 10,593,124,352 bytes。該快取最後一般檔修改於 2026-05-07；三個原始 ZIP 指向公開的 `DaJhuan/ICCAD`，2026-09-30 HEAD 皆回 200，Content-Length 與本機 `dataset_info.json` 的 `num_bytes` 三筆完全一致。主要內容原為 6.32 GB Arrow 處理快取與 4.16 GB 下載／解壓快取；本次 `/proc` cwd/root/fd 未見程序引用、無子掛載。使用者確認後，已刪除全部 12 個 `.arrow` 檔，`df -B1 /` 可用空間增加 6,321,922,048 bytes，複查 `.arrow` 為 0；下載／解壓與模型快取保留。重新使用資料集時須重新產生 Arrow 處理快取。詳見 Zeus `<admin-home>/playground/storage-audit-20260903/fleet-remaining-large-20260928/ATHENA-DISK-20260930.md`。
