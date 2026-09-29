---
title: NFS jack0716 home storage inventory
scope: machines/hapi-fleet
status: active
updated: 2026-09-30
---

# NFS `jack0716` home 容量與 EDA 內容

- 2026-09-30 使用者確認先前所稱 `jack0314` 指的是 `jack0716`；NIS 和 NFS 沒有 `jack0314` 帳號。
- `<remote-home>/jack0716/3dic` 的 NFS 完整 `du -x -B1` 為 **637,871,976,448 bytes**：3D-IC 研究樹、Chipyard／OpenROAD 工作樹及大量 `research/output` 參數掃描結果。`<remote-home>/jack0716/tsri.retired-20260914` 獨立完整 `du` 為 **445,924,265,984 bytes**，包含 VCS、Verdi、Design Compiler、ICC2、SpyGlass、VC Formal、Innovus、JasperGold、3DIC Compiler 與 cell／製程庫的實際安裝檔。兩樹合計 **1,083,796,242,432 bytes（1.084 TB）**，尚不含 `.cache` 29.68 GB、`Socv_TA` 18.34 GB 等其他目錄；不是全帳號總量。
- `tsri.retired-20260914/eda.bashrc` 仍指向不存在的舊 `<remote-home>/jack0716/tsri`，頂層登入設定未見引用；只證明舊啟動路徑失效，不能直接推論整棵 EDA 軟體／製程庫可刪。本次僅讀取，未改動 home。詳細回執見 Zeus `<admin-home>/playground/storage-audit-20260903/fleet-remaining-large-20260928/FLEET-DISK-RECHECK-20260929.md`。
- 2026-09-30 再查 EDA 維運紀錄：`tsri` 於 09-14 改名退休，當時已把同版工具複製至共用 `/apps/cad`，後續發布至 `/apps/eda`。兩樹 19 個頂層名稱／類型相同；VCS `vcs1` 與 IC Contest cell library `typical.db` 的舊樹／`/apps/cad` SHA-256 各自一致。舊樹 VCS inode 與 `/apps/cad` 不同、`nlink=1`；`/apps/cad` 與正式 `/apps/eda` 同一 inode、`nlink=2`。Zeus、Mazu、Cthulhu、Valkyrie 的 `/proc` cwd/root/fd 與運作 Docker 掛載未見退休路徑；Valkyrie 舊停止容器 `opentitan_bug8729` 掛的是已不存在的 `<remote-home>/jack0716/tsri`。**現行工具運作不依賴退休樹**；未對數百萬檔做完整 bit-exact 比對，不能宣稱整棵樹無任何獨有小檔。詳細見 Zeus 上述盤點紀錄。
