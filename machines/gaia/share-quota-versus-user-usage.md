---
title: "Share quota 不等於每個 NIS 使用者的用量"
scope: machines/gaia
status: active
updated: 2026-09-16
evidence_digest: 0b3bf56c34c23f5a7cf86d4e42959d1ec7a57a7930468e02fb961f781ea81f2e
---

# Share quota 不等於每個 NIS 使用者的用量

原始 DSM 回應提供共享資料夾 quota 值，數個 user-quota 探測則失敗；client 對他人 home 執行 du 出現 Permission denied，本人 home 的 NFS 掃描也耗時。

分開核對 share、UID／帳號與量測權限，失敗或部分可讀的 du 不可當完整每人用量。這些回應不證明 DSM 永遠無法對接 NIS、一定有 root squash 或只有 root du 唯一路徑；排程本機快照只是當次提案，沒有建立或輸出驗證。
