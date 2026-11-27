---
title: cthulhu 與 mazu 的 NFSv4.1 TEST_STATEID 風暴
scope: machines/cthulhu
machine: [cthulhu, mazu]
status: cthulhu recovered on 2026-11-19, mazu recurrence confirmed
updated: 2026-11-28
tags: [nfs, nfsv4.1, delegation, test_stateid, kernel-module, synology]
---

# cthulhu 與 mazu 的 NFSv4.1 TEST_STATEID 風暴

mazu 的 HAPI Codex 再出現 `startup is still unconfirmed` 時，先從兩次 `/proc/self/mountstats` 算出 `TEST_STATEID` 每秒速率與 `GETATTR` 平均排隊、往返時間。若大量時間花在排隊，用 `nfs4:nfs4_sequence_done` 按 `status_flags` 和 `error` 計數，確認 NAS 是否持續回報 `0x40`。mazu 的一般帳號可讀自己的 mountstats。讀核心 tracepoint 需要管理權限。

**症狀**：cthulhu load 很高，但 CPU 不忙；大量程序卡在 D state 等 NFS。網路和 NAS 硬碟都不是瓶頸。

**根因**：NAS（Synology，NFSv4.1）在 SEQUENCE 回覆裡設了 `SEQ4_STATUS_RECALLABLE_STATE_REVOKED`（0x40），client 收到後對所有 delegation stateid 狂發 `TEST_STATEID`，約 5,000 次/秒，永遠清不掉這個旗標。可從 `/proc/self/mountstats` 看到 RPC 數量：2026-11-18 `/home` 累計 3.2 億次 TEST_STATEID、3,800 萬次 DELEGRETURN，證明 delegation 開著且大量使用。

**修法**：上游 commit `220af23d8639`（"NFS: Return a delegation the client fails to record"）。**Ubuntu 26.04 的官方 kernel `7.0.0-43-generic`（7.0.0-43.43）已經收進這個修正**，changelog 和原始碼都可核對。這修正了特定用戶端漏還 delegation 的路徑。mazu 後來在同一版本復發。athena 跑 -43 時 delegation 照常使用，TEST_STATEID 為 0。官方 kernel 有 Canonical 簽章，開著 Secure Boot 的 mazu 也能用。

2026-11-18 一開始自編了 -38 用的 `nfsv4.ko`，加上開機保護，後來發現 -43 已經修好，就全部移除了。升級時要注意：
- 機群都關掉了自動更新，所以各台 kernel 版本會不一樣，看各自上次手動 apt 是什麼時候。
- `apt full-upgrade` 裝了新 kernel，但**沒有幫 DKMS 的 `nvidia-srv` 編新版**（cthulhu 就發生過），重開前一定要確認 `modinfo -k <kver> -n nvidia` 找得到。2026-11-19 起全部改用 Canonical 預編模組，新 kernel 會自動帶對應模組；驅動版本見 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]]。
- zeus、valkyrie、mazu 的 apt 清單太舊，要先 `apt-get update`，才看得到新 kernel。

**NAS 端**：DSM 的 NFS 設定介面和 API 都沒有關 delegation 的選項。只能 SSH 進 NAS 執行 `echo 0 > /proc/sys/fs/leases-enable`，但不持久，而且影響所有 client，所以不建議。

狀態：cthulhu、zeus、valkyrie、mazu、athena、utux都用 `shutdown -r` 排在 2026-11-19 09:30 重開（不等使用者的工作；原本 08:00 的 reboot-when-idle timer 已移除）。重開後用 `/proc/self/mountstats` 確認 TEST_STATEID 降到接近 0。

相關：[[machines/hapi-fleet/storage-migration-2026]]、[[machines/hapi-fleet/system-install-via-admin]]

**2026-11-19 重開後驗證**：cthulhu、mazu、zeus、valkyrie、athena 全部跑 `7.0.0-43-generic`。`/proc/self/mountstats`（用 `su_<host>`／swear02 才有 sudo，swear01 沒有）的 TEST_STATEID 五台都是 0，10 秒取樣也是 0/s；DELEGRETURN 不是 0，代表 delegation 仍在正常使用。cthulhu 開機 2.5 小時內 OPEN 約 20 萬次也沒有風暴。D state 只剩使用者 `cp` 正常等 I/O；dmesg 只有一筆 `fileid changed`，不是這個 bug。

**2026-11-27 復發（mazu）**：mazu 跑的仍是 7.0.0-43，10 秒取樣 `/home` 的 TEST_STATEID 約 1,131 次/秒，同時段 valkyrie 為 0。`GETATTR` 排隊 18 ms，HAPI 的 Codex 啟動因此超過 20 秒期限，見 [[tools/hapi/codex-fleet-peer-protocol]]。

## mazu 復發的實測與判讀（2026-11-28）

- `/home` 的 10 秒取樣約每秒 4,312 次 `TEST_STATEID`，`GETATTR` 平均在用戶端排隊 125 ms，網路往返 0.23 ms。兩次獨立核心取樣中，3 秒的 14,932 筆與 7 秒的 161 筆成功 `SEQUENCE` 回覆，旗標全部只有 `SEQ4_STATUS_RECALLABLE_STATE_REVOKED`（0x40）。後一次的 159 次 `TEST_STATEID` 都回報該 stateid 仍有效。協定要求用戶端找到被撤銷的物件後用 `FREE_STATEID` 確認。取樣期間未找到該物件。[RFC 8881 §18.46.3](https://www.rfc-editor.org/rfc/rfc8881.html#section-18.46.3)
- Ubuntu `7.0.0-43.43` 已含用戶端修正 `220af23d`。`-44.44` 的六個相關 NFS 用戶端檔案與 `-43.43` 相同。Linux NFSD 另修過伺服器保留一筆用戶端不認得的 revoked delegation、導致旗標持續存在的問題。這與取樣相符。是否為同一缺陷仍需 NAS 的 callback 回覆和伺服器狀態才能確認。[用戶端修正](https://github.com/torvalds/linux/commit/220af23d8639) · [伺服器修正](https://github.com/torvalds/linux/commit/bbfa823f624d)
- mazu 短暫將 `delegation_watermark` 從 5000 調到 32 時，原生 Codex 0.172.0 的 `thread/start` 仍耗費 86.54 秒，之後已恢復 5000。HAPI v0.36.7.1 的 shared runtime 對這個 RPC 等 20 秒。判定處理有效應用正式 Runner 新開 session 並收到模型回覆，僅看到 RPC 速率下降不足以通過驗證。[HAPI 呼叫位置](https://github.com/swear01/hapi/blob/v0.36.7.1/cli/src/codex/shared/runtime.ts#L248-L254) · [RPC 時限](https://github.com/swear01/hapi/blob/v0.36.7.1/cli/src/codex/codexAppServerClient.ts#L348-L350)
