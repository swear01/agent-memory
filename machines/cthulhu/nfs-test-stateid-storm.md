---
title: cthulhu NFS 卡頓：NFSv4.1 client 的 TEST_STATEID 風暴與 patch
scope: machines/cthulhu
machine: cthulhu
status: patch installed 2026-11-18, reboot 2026-11-19 05:00 pending verification
updated: 2026-11-18
tags: [nfs, nfsv4.1, delegation, test_stateid, kernel-module, synology]
---

# cthulhu NFS 卡頓：TEST_STATEID 風暴

**症狀**：cthulhu load 很高，但 CPU 不忙；大量程序卡在 D state 等 NFS。網路和 NAS 硬碟都不是瓶頸。

**根因**：NAS（Synology，NFSv4.1）在 SEQUENCE 回覆裡設了 `SEQ4_STATUS_RECALLABLE_STATE_REVOKED`（0x40），client 收到後對所有 delegation stateid 狂發 `TEST_STATEID`，約 5,000 次/秒，永遠清不掉這個旗標。看 `/proc/self/mountstats`（要 sudo）的 `TEST_STATEID` / `DELEGRETURN` 計數就能確認：2026-11-18 `/home` 累計 3.2 億次 TEST_STATEID、3,800 萬次 DELEGRETURN，證明 delegation 開著且大量使用。

**修法**：主線 commit `220af23d8639` 修了這個 client 迴圈。已編成 out-of-tree `nfsv4.ko`，給 kernel `7.0.0-38-generic`，裝在 `/lib/modules/7.0.0-38-generic/updates/nfsv4.ko.zst`。另裝了開機保護 `/usr/local/sbin/nfsv4-patch-guard`，配 `nfsv4-patch-guard` / `nfsv4-patch-confirm` 兩個 unit：新模組開機出事就自動退回原版。**換 kernel 版本後 patch 不會跟著走**，要重編。

**NAS 端**：DSM 的 NFS 設定介面和 API 都沒有關 delegation 的選項。唯一做法是 SSH 進 NAS 執行 `echo 0 > /proc/sys/fs/leases-enable`，但這不持久、DSM 升級可能蓋掉、而且影響所有 client，所以沒做。client patch 無效時才考慮。

待辦：2026-11-19 05:00 重開後確認 TEST_STATEID 速率降下來；有效再推到 valkyrie、mazu、athena 等其他掛同一 NAS 的主機。建置用的 qemu 套件和 `/var/tmp/nfsvmtest` 可以清掉。

相關：[[machines/cthulhu/shared-nfs-home]]、[[machines/hapi-fleet/system-install-via-admin]]
