---
title: "Issue25 流程換版須一併退役中間產物"
scope: projects/issue25-memory-promotion
status: active
updated: 2026-09-28
evidence_digest: c8e28f9f5a98bd013a20c6b92548c75a72195554dbea5ab497015f64d5e35abb
---

# Issue25 流程換版須一併退役中間產物

控制者切換到訊息投影與完整佇列後，仍留下舊 assembled/shard 資料庫、展開批次、草稿與逐 PR 輔助腳本。使用者要求清理時，退役產物合計約 531.8 GB；本機還有 326 份中間檔。來源保留的要求不能被解讀成所有可重建的工作副本都必須永久堆放。

流程換版或完成時，把原始來源、唯一模型答案、來源綁定及最終處置帳冊，與可重建的中間產物分開。先確認現行流程不再依賴待刪路徑、沒有 worker 或開啟檔案仍在使用，再核對檔案歸屬、非 symlink 與保留來源。退役工作產物依明確用途與相依關係刪除，不以相同名稱或大小宣稱逐位元重複；唯一來源的刪除不能沿用這個依據。大型退役批次可保留原有 hash manifest 與清理憑據，毋須為了刪除而重跑整套模型或重算現用大索引。

本次刪後確認路徑不存在，移除兩個 staging 目錄共 1,928 份 Markdown，保留其結構化原稿與來源帳冊作稽核；本機工作目錄只保留最後憑據。清理同時涵蓋自己的已合併分支、乾淨 worktree、過期監督器與歷史失敗狀態，沒有停止正式索引服務或刪除無關 dirty worktree。這是一次性工作產物的生命周期收尾，不是全機快取或原始資料的一般刪除授權。

2026-09-28 後續：使用者確認替代檢索驗證已完成後，Cthulhu 兩棵退役資料樹精確刪除：舊 QMD index 228,296,683,520 bytes、已搬存的 Athena 無效分片 133,587,517,440 bytes，系統碟合計釋放 361,884,200,960 bytes。此次未重跑檢索驗證；刪除前確認無程序 fd/maps/cwd/root/exe、子掛載、systemd unit 或掃描範圍內設定引用。保留設定與 verified/passed 報告、manifest 等 36 項小型憑據於管理員私有 `<remote-home>/private-system-backups/issue25-cthulhu-retired-20260928/evidence.tar.zst`，SHA-256 `47ffe6f4d742bdca41181cfcb3f7e32aeaf1b80df4085261713544c36ec3deaf`；原始工作來源未動。先前「舊索引作 rollback 直到 retrieval gate」的條件已由使用者確認解除，勿再將兩棵路徑列為待保留資料。
