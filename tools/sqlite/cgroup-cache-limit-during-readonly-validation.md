---
title: "大型 SQLite 唯讀驗證也會受 cgroup 檔案快取上限拖慢"
scope: tools/sqlite
status: active
updated: 2026-09-16
---

# 大型 SQLite 唯讀驗證也會受 cgroup 檔案快取上限拖慢

本次 G16 驗證的 SQLite 資料庫為 83.6 GB，服務 MemoryMax 卻只有 32 GiB。檢查時 memory.current 已到上限，約 34.08 GB 為 file cache，anon 約 86 MB；memory.events 出現大量 max 命中、workingset_refault_file 約 2,821 萬頁，而 oom 與 oom_kill 都是零。程序持續前進但大量等待磁碟，不能只憑 RSS 很低或沒有 OOM 就排除記憶體限制。

先查看該服務 cgroup 的 memory.current、memory.stat、memory.events 與主機 MemAvailable，分清匿名記憶體和檔案快取。這次主機約有 117 GiB 可用，因此只將現行驗證服務的 MemoryMax 提高到 96 GiB，留下主機餘裕；不改 CPU 80%、熱保護、資料庫或驗證內容，也不重啟已在跑的程序。

調整後同一 PID 的快取可繼續成長，第一個獨立程序通過完整 SHA、SQLite quick_check、三種筆數、缺漏與雙向對帳以及 KNN canaries，WAL 仍為零。第二輪與最終檢索結果須另看完成報告；這項觀察沒有量出全部耗時的因果占比，不保證固定加速倍數，也不把 96 GiB 當作其他主機的通用值。
