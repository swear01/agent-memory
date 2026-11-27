---
title: cthulhu NFS 卡頓：NFSv4.1 client 的 TEST_STATEID 風暴（kernel 7.0.0-43 已修）
scope: machines/cthulhu
machine: cthulhu
status: fixed — verified 2026-11-19 after reboot
updated: 2026-11-27
tags: [nfs, nfsv4.1, delegation, test_stateid, kernel-module, synology]
---

# cthulhu NFS 卡頓：TEST_STATEID 風暴

**症狀**：cthulhu load 很高，但 CPU 不忙；大量程序卡在 D state 等 NFS。網路和 NAS 硬碟都不是瓶頸。

**根因**：NAS（Synology，NFSv4.1）在 SEQUENCE 回覆裡設了 `SEQ4_STATUS_RECALLABLE_STATE_REVOKED`（0x40），client 收到後對所有 delegation stateid 狂發 `TEST_STATEID`，約 5,000 次/秒，永遠清不掉這個旗標。看 `/proc/self/mountstats`（要 sudo）的 `TEST_STATEID` / `DELEGRETURN` 計數就能確認：2026-11-18 `/home` 累計 3.2 億次 TEST_STATEID、3,800 萬次 DELEGRETURN，證明 delegation 開著且大量使用。

**修法**：上游 commit `220af23d8639`（"NFS: Return a delegation the client fails to record"）。**Ubuntu 26.04 的官方 kernel `7.0.0-43-generic`（7.0.0-43.43）已經收進這個修正**，changelog 和原始碼都有，所以直接升級 kernel 就好，不需要自編模組。athena 跑 -43 時 delegation 照常使用，TEST_STATEID 為 0。官方 kernel 有 Canonical 簽章，開著 Secure Boot 的 mazu 也能用。

2026-11-18 一開始自編了 -38 用的 `nfsv4.ko`，加上開機保護，後來發現 -43 已經修好，就全部移除了。升級時要注意：
- 機群都關掉了自動更新，所以各台 kernel 版本會不一樣，看各自上次手動 apt 是什麼時候。
- `apt full-upgrade` 裝了新 kernel，但**沒有幫 DKMS 的 `nvidia-srv` 編新版**（cthulhu 就發生過），重開前一定要確認 `modinfo -k <kver> -n nvidia` 找得到。2026-11-19 起全部改用 Canonical 預編模組，新 kernel 會自動帶對應模組；驅動版本見 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]]。
- zeus、valkyrie、mazu 的 apt 清單太舊，要先 `apt-get update`，才看得到新 kernel。

**NAS 端**：DSM 的 NFS 設定介面和 API 都沒有關 delegation 的選項。只能 SSH 進 NAS 執行 `echo 0 > /proc/sys/fs/leases-enable`，但不持久，而且影響所有 client，所以不建議。

狀態：cthulhu、zeus、valkyrie、mazu、athena、utux都用 `shutdown -r` 排在 2026-11-19 09:30 重開（不等使用者的工作；原本 08:00 的 reboot-when-idle timer 已移除）。重開後用 `/proc/self/mountstats` 確認 TEST_STATEID 降到接近 0。

相關：[[machines/hapi-fleet/storage-migration-2026]]、[[machines/hapi-fleet/system-install-via-admin]]

**2026-11-19 重開後驗證**：cthulhu、mazu、zeus、valkyrie、athena 全部跑 `7.0.0-43-generic`。`/proc/self/mountstats`（用 `su_<host>`／swear02 才有 sudo，swear01 沒有）的 TEST_STATEID 五台都是 0，10 秒取樣也是 0/s；DELEGRETURN 不是 0，代表 delegation 仍在正常使用。cthulhu 開機 2.5 小時內 OPEN 約 20 萬次也沒有風暴。D state 只剩使用者 `cp` 正常等 I/O；dmesg 只有一筆 `fileid changed`，不是這個 bug。

**2026-11-27 復發（mazu）**：mazu 跑的仍是 7.0.0-43，10 秒取樣 `/home` 的 TEST_STATEID 約 1,131 次/秒，同時段 valkyrie 為 0。`GETATTR` 排隊 18 ms，HAPI 的 Codex 啟動因此超過 20 秒期限，見 [[tools/hapi/codex-fleet-peer-protocol]]。觸發原因尚未查明，下一步是比對 mazu 與 NAS 的 delegation 狀態。
