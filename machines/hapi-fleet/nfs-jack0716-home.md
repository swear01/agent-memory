---
title: NFS jack0716 home storage inventory
scope: machines/hapi-fleet
status: active
updated: 2026-09-29
---

# NFS `jack0716` home 容量與 EDA 內容

- 2026-09-29 使用者詢問 `jack0314` 的 1 TB 多資料；目前 NIS 和 NFS 找不到 `jack0314`，只有 `jack0716`，帳號指認待確認。**以下數字僅屬 `jack0716`，不可當成 `jack0314` 的盤點。**
- `<remote-home>/jack0716/3dic` 的 NFS 完整 `du -x -B1` 為 **637,871,976,448 bytes**：3D-IC 研究樹、Chipyard／OpenROAD 工作樹及大量 `research/output` 參數掃描結果。`<remote-home>/jack0716/tsri.retired-20260914` 獨立完整 `du` 為 **445,924,265,984 bytes**，包含 VCS、Verdi、Design Compiler、ICC2、SpyGlass、VC Formal、Innovus、JasperGold、3DIC Compiler 與 cell／製程庫的實際安裝檔。兩樹合計 **1,083,796,242,432 bytes（1.084 TB）**，尚不含 `.cache` 29.68 GB、`Socv_TA` 18.34 GB 等其他目錄；不是全帳號總量。
- `tsri.retired-20260914/eda.bashrc` 仍指向不存在的舊 `<remote-home>/jack0716/tsri`，頂層登入設定未見引用；只證明舊啟動路徑失效，不能直接推論整棵 EDA 軟體／製程庫可刪。本次僅讀取，未改動 home。詳細回執見 Zeus `<admin-home>/playground/storage-audit-20260903/fleet-remaining-large-20260928/FLEET-DISK-RECHECK-20260929.md`。
