---
title: cthulhu 與 mazu 的 NFSv4.1 TEST_STATEID 風暴
scope: machines/cthulhu
machine: [cthulhu, mazu]
status: cthulhu recovered on 2026-11-19, mazu recurrence confirmed
updated: 2026-11-28
tags: [nfs, nfsv4.1, delegation, test_stateid, kernel-module, synology]
---

若 load 升高、CPU 不忙、大量程序在 D state 等 NFS，或 mazu 的 HAPI Codex 出現 `startup is still unconfirmed`，用兩次 `/proc/self/mountstats` 比較 `/home` 的 `TEST_STATEID` 每秒速率，以及 `GETATTR` 排隊與往返時間。一般帳號可讀自己的 mountstats。排隊明顯偏高時，由有權限的管理者讀 `nfs4:nfs4_sequence_done`，按 `status_flags` 和 `error` 計數。

- mazu 復發時成功的 SEQUENCE 全帶 `0x40 SEQ4_STATUS_RECALLABLE_STATE_REVOKED`，但被取樣的 TEST_STATEID 都仍有效。找到 revoked stateid 後才可用 FREE_STATEID 確認。Linux NFSD 有能解釋此現象的伺服器修正。要確認 NAS 是同一缺陷，需查 callback 回覆與伺服器的 delegation 狀態。避免全域停用 NAS leases，這會影響所有 client。[RFC 8881 §18.46.3](https://www.rfc-editor.org/rfc/rfc8881.html#section-18.46.3) · [NFSD 修正](https://github.com/torvalds/linux/commit/bbfa823f624d)
- Ubuntu 官方簽章 kernel `7.0.0-43.43` 已含用戶端修正 `220af23d8639`，取代已撤除的 -38 自編模組。cthulhu 重開後 TEST_STATEID 歸零且 delegation 仍生效。mazu 同版仍復發，`-44.44` 的六個相關用戶端檔案相同。換 kernel 前核對實際 apt 版本與 `modinfo -k <kver> -n nvidia`，參考 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]] 和 [[machines/hapi-fleet/storage-migration-2026]]。[用戶端修正](https://github.com/torvalds/linux/commit/220af23d8639)
- mazu 短暫降低 `delegation_watermark` 後，`thread/start` 仍超過 HAPI v0.36.7.1 的 20 秒等待時間，設定已還原。驗證須從正式 Runner 開新 session 並收到模型回覆，同時複查 RPC 速率及排隊時間。參考 [[tools/hapi/codex-fleet-peer-protocol]]。[HAPI RPC 時限](https://github.com/swear01/hapi/blob/v0.36.7.1/cli/src/codex/codexAppServerClient.ts#L348-L350)

[逐台 kernel、回覆旗標與時序取樣](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/cthulhu/nfs-test-stateid-storm.md)
