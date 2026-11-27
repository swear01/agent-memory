---
title: cthulhu 與 mazu 的 NFSv4.1 TEST_STATEID 風暴
scope: machines/cthulhu
machine: [cthulhu, mazu]
status: mazu production mounts use NFSv4.0, cause of prior NFSv4.1 recurrence unconfirmed
updated: 2026-11-28
tags: [nfs, nfsv4.1, delegation, test_stateid, kernel-module, synology]
---

2026-11-28 經授權讓 mazu 正常關機重啟，`/etc/fstab` 將正式 `/home`、`/apps` 都指定為同一批 NAS export 的 `vers=4.0`。這是整台 mazu 的設定。重開後以 `findmnt -t nfs,nfs4` 核對兩個實際掛載，在正式 `/home` 通過寫入、fsync、rename、重開讀回，以及獨立程序的鎖衝突與釋放測試。`/apps` 可讀一般檔案，兩個正式掛載的 `TEST_STATEID` 累計均為 0。NFSv4.0 本身不送 NFSv4.1 的這項操作。

- 回到 v4.1 前，先比對 `/var/backups/mazu-nfs-cutover-2rs3ymws/fstab.original` 與現行 fstab，保留其後其他變更。安排維護時段排空作業、寫入與檔案鎖，還原兩個掛載的原設定，經正常停機切換後再驗讀寫、鎖與 RPC。修改 fstab 不會改變正在使用的掛載版本。
- 同一台 mazu 重開後另外以正常 NFSv4.1 掛兩個 export 做對照。28.1 秒內讀取小檔 3,806 次，讀寫、fsync、rename 與鎖測試通過。349 筆成功 `SEQUENCE` 的旗標全為 0，兩個測試掛載的 `TEST_STATEID` 也全為 0。這只證明新建立的 v4.1 client state 在該次短測正常，後續是否復發仍須觀察。

- Gaia 升至 DSM 7.4.1-90080 Update 2 後，在 mazu 重跑相同的獨立 v4.1 掛載測試，28.6 秒讀取 3,762 次，寫入、fsync、rename 與跨程序鎖皆通過，349 筆成功 SEQUENCE 的旗標均為零，兩個 export 的 TEST_STATEID 增量也都是零。舊韌體在全新連線狀態已通過同一測試，這項對照無法證明韌體消除了長時間運行後的風暴。更新與用戶端復原流程見 [[machines/gaia/dsm-api-apps-share]]。

若 load 升高、CPU 不忙、大量程序在 D state 等 NFS，或 mazu 的 HAPI Codex 出現 `startup is still unconfirmed`，用兩次 `/proc/self/mountstats` 比較 `/home` 的 `TEST_STATEID` 每秒速率，以及 `GETATTR` 排隊與往返時間。一般帳號可讀自己的 mountstats。排隊明顯偏高時，由有權限的管理者讀 `nfs4:nfs4_sequence_done`，按 `status_flags` 和 `error` 計數。

- mazu 復發時成功的 SEQUENCE 全帶 `0x40 SEQ4_STATUS_RECALLABLE_STATE_REVOKED`，但被取樣的 TEST_STATEID 都仍有效。找到 revoked stateid 後才可用 FREE_STATEID 確認。Linux NFSD 有能解釋此現象的伺服器修正。要確認 NAS 是同一缺陷，需查 callback 回覆與伺服器的 delegation 狀態。避免全域停用 NAS leases，這會影響所有 client。[RFC 8881 §18.46.3](https://www.rfc-editor.org/rfc/rfc8881.html#section-18.46.3) · [NFSD 修正](https://github.com/torvalds/linux/commit/bbfa823f624d)
- 故障時 Gaia 經 DSM 確認是 DS1821+、`7.2.1-69057 Update 13`。Synology 公開的 V1000 GPL 核心基線（7.2-64570、7.2-72806、7.3-86009、7.4-90080）均為 Linux 4.4.302。所查 NFSD 原始碼仍保留 revoked delegation 清單與舊 recall 處理，未見上游 `bbfa823f624d` 對已拒絕 recall 的處理。基線原始碼不能證明特定 Update 的執行中模組內容，原廠也未公開確認能修此症狀的 DSM build。若要正式恢復 NFSv4.1，請 Synology 對照該修正、callback 回覆與 server state，確認支援的修正版。[Synology GPL 原始碼](https://archive.synology.com/download/ToolChain/Synology%20NAS%20GPL%20Source) · [DS1821+ 發行說明](https://www.synology.com/en-us/releaseNote/DSM?model=DS1821%2B)
- Ubuntu 官方簽章 kernel `7.0.0-43.43` 已含用戶端修正 `220af23d8639`，取代已撤除的 -38 自編模組。cthulhu 重開後 TEST_STATEID 歸零且 delegation 仍生效。mazu 同版仍復發，`-44.44` 的六個相關用戶端檔案相同。換 kernel 前核對實際 apt 版本與 `modinfo -k <kver> -n nvidia`，參考 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]] 和 [[machines/hapi-fleet/storage-migration-2026]]。[用戶端修正](https://github.com/torvalds/linux/commit/220af23d8639)
- mazu 短暫降低 `delegation_watermark` 後，`thread/start` 仍超過 HAPI v0.36.7.1 的 20 秒等待時間，設定已還原。驗證須從正式 Runner 開新 session 並收到模型回覆，同時複查 RPC 速率及排隊時間。參考 [[tools/hapi/codex-fleet-peer-protocol]]。[HAPI RPC 時限](https://github.com/swear01/hapi/blob/v0.36.7.1/cli/src/codex/codexAppServerClient.ts#L348-L350)

[逐台 kernel、回覆旗標與時序取樣](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/cthulhu/nfs-test-stateid-storm.md)
