---
title: DVLab fleet persistent storage migration inventory
scope: machines/hapi-fleet
project: dvlab-storage
tags: [storage, migration, backup, cleanup, fleet]
status: active
created: 2026-08-26
updated: 2026-08-26
---

# DVLab persistent storage migration inventory

## 已驗證的暫存邊界

- Mazu、Cthulhu、Athena、Valkyrie、Zeus 的 `/tmp` 都是 tmpfs，重開機會清空。
- 五台的 `/var/tmp` 都位於 root ext4，重開機不會清空；`systemd-tmpfiles` 設為 30 天 age-based 清理，clean timer 為 active。
- `/var/tmp` 可能包含現役 HAPI、Codex、OpenCode、Cursor 與研究工作狀態。不要人工整批清理，應交由 age policy 與服務本身管理。

## 2026-08-26 舊搬家世代快照

下列數字是盤點快照，執行刪除前必須重新量測：

| 世代 | 位置 | 快照容量 | 性質 |
|---|---|---:|---|
| G1 | Valkyrie `/mnt/md1` | 5.563 TiB 已用 | 舊 Cthulhu 家目錄世代 |
| G2 | Zeus `<remote-home>.bak` | 1.202 TiB 已用 | 2024-03 前後的舊家目錄世代 |
| G3 | NAS `<remote-home>/.backup` | 1.194 TiB | 2024-03 的完整 home tar |
| G4 | NAS `<remote-home>/copy/cthulhu_home` | 260.06 GiB | 2025-05 的搬家 tgz |

排除現行共享 `<remote-home>`，四代合計約 8.213 TiB。G1 逐帳號可讀內容為 5.300 TiB，其中約 4.012 TiB 是條件式排除候選；若全數通過刪除門檻，G1 可讀內容約降至 1.288 TiB，才有機會讓冷備份總量低於 2 TiB。

候選類別包括衍生 log、可重取 benchmark/testcase 副本、嵌套舊備份、課程影片 dataset、`.vscode-server` 與 Conda package cache。不能只因為檔案年代久遠就判定可刪。

## 刪除門檻

每一批都要同時完成：

1. 重新解析掛載點與裝置，避免誤碰現行共享 `<remote-home>` 或冷備份碟。
2. 以 `fuser` 或 `lsof` 確認沒有程序使用。
3. 保存檔案清單、容量、mtime、owner；重要研究封存另存 SHA256。
4. 抽取 README、論文、報告、原始碼、腳本、最終結果及不可重建輸入。
5. 對可下載 dataset、模型、套件與 benchmark 記錄來源或版本。
6. 冷備份碟連接後完成實際讀回、解壓與 hash 驗證。
7. 先清可再生成資料，再清連續搬家世代，最後才整代退役；每批以 `df` 核對釋放量。
8. 權限不可讀與 `df`/`du` 差額必須由 root 級盤點補完。

## 目前狀態與文件邊界

- 盤點時沒有執行 persistent 資料、舊搬家世代或 `/var/tmp` 的刪除。
- 5TB 冷備份碟尚未連接，因此沒有執行搬移或還原測試。
- 詳細逐帳號清單保留在任務盤點文件，不複製進 shared memory。
- 個人 Drive 上的 Google 文件只是暫時閱讀副本，不是永久 canonical 紀錄；memory 不保存帳號、Google file ID 或受限文件 URL。
