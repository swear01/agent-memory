---
title: "跨來源磁碟盤點不要只用 mountpoint 串接"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: 9afa6b5d90f19046862f518a9c5401776d7d17763452ee2eb315d876b8e843b3
---

# 跨來源磁碟盤點不要只用 mountpoint 串接

歷史助手回報系統碟在合併清單中消失：Glances 保留根掛載點，lsblk 卻回報同一裝置的 bind-mount 路徑，字串 join 因而落空。

先確認兩來源的裝置身分與一對多掛載關係，再合併盤點。當次提案是正規化 device 名稱，不依賴 mountpoint 相等；裝置名称也可能隨開機或命名空間改變，不能把它當永久識別碼。來源沒有修正後清單驗證。
