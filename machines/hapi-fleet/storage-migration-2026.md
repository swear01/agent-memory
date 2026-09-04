---
title: DVLab fleet persistent storage migration inventory
scope: machines/hapi-fleet
project: dvlab-storage
tags: [storage, migration, backup, cleanup, fleet]
status: active
created: 2026-08-26
updated: 2026-09-04
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

## G2 精確重複的判讀（2026-09-04）

- G2 大於 1 MiB 的檔案有 12,402 組 SHA-256 精確重複、27,431 個多餘路徑；表觀理論可回收 267.816 GB，其中跨帳號 160.978 GB。
- `stat` inode 回查證明其中 57.487 GB 原本已是 hard link，已共用實體空間。重複報表必須先按 `(st_dev, st_ino)` 去重，不能把表觀重複量直接當成 `df` 可釋放量。
- 跨帳號相同內容常見於公開 dataset、課程 testcase、Conda/CUDA library 與 editor runtime。直接刪掉某一路徑會破壞該帳號封存目錄的完整性；ext4 hard link 又無法保留不同 owner，因此必須先選 canonical copy，或整批淘汰可重建環境。
- G2 中可整批重建的頂層 cache、Conda/Anaconda/Micromamba、VS Code Server 與 Snap 共 97 個目錄、451.53 GiB。這類應以完整目錄為刪除單位；不要逐檔刪除重複 library 後留下半壞環境。`.local` 不可直接列入，裡面可能有使用者腳本或唯一資料。

## G1 與 G2 的跨世代精確比對（2026-09-04）

- 先以帳號內相對路徑與大小篩出 20,908 個候選、486.51 GiB，再做雙邊 SHA-256；20,798 個、485.87 GiB 相同，另 110 個、0.637 GiB 即使路徑與大小相同，內容仍不同。尺寸與路徑只能當候選條件，不能當刪除證明。
- 跨世代相同檔案採 G2 為 canonical copy、G1 為 delete-later path；詳細 pair manifest 的 SHA-256 為 `d9600680eeb00bfa412586216bebd6f37f894208d2975209ee52dfb5c924473a`。
- G1 的 20,798 個路徑實際只有 18,390 個唯一 inode。以 `(st_dev, st_ino, st_nlink)` 校正並確認清單涵蓋所有 hard-link 名稱後，預估可釋放量由表觀 485.87 GiB 降為 441.79 GiB。
- 被共享 home 遮住的 Valkyrie 本地 home 另有 7,361 個與 G2 相同的路徑，表觀 125.17 GiB；實際為 4,632 個唯一 inode、83.77 GiB。配對 manifest SHA-256 為 `2e9fd9f28b478cefec2cd880b861023e03f66ddcb4bb792bc6230dda8b26bf50`。
- 因此跨檔案系統 hash 相同也不能直接把檔案大小加總當成 `df` 回收量；刪除端仍須先以 inode/link count 去重，並確認沒有清單外 hard link。

## G1 深層容量分級（2026-09-04）

- G1 的 `du` 使用量為 5.563 TiB。跨 G2 exact copy 與 124 個嚴格可重建根目錄去除重疊後為 656.84 GiB；再加上一個藏在舊備份內、經目錄結構確認的 16.44 GiB Anaconda 環境，直接清理層約 673.28 GiB（11.82%）。
- 四個冷帳號已定位 3.610 TiB（64.89%）的批次 log、proof trace 與 testcase output。這一層不是整個帳號直接刪除：先抽出 source、論文、重現腳本、final/best 結果與摘要，再刪或壓縮原始輸出。
- 最大帳號有四個未壓縮 tar，合計 1.700 TiB；完整 member inventory 共 3,686 個 regular files，全部為 `.log`、沒有 non-log。相鄰的另一個大型 tar 混有 source、文件、結果與 Git metadata，必須分開 HOLD，不能用檔名或同目錄位置推論內容。
- `du` 與表觀檔案大小都要保留：某個訓練 `tmp` 目錄表觀約 87 GiB、實際配置僅 12.93 GiB，因 sparse files 不能把 `st_size` 當成回收容量。
- G1 內部大於等於 1 GiB 的同大小候選，尺寸法推測 79.41 GiB 重複；完整 SHA-256 後只剩 16 組、43.21 GiB。扣除既有 exact/rebuildable 與四大輸出區後，新增精確資料重複約 6.52 GiB，仍須先指定 canonical owner。另有同檔名同大小但雜湊不同的 benchmark input，明確不可刪。
- 巨型文字輸出的 zstd level-1 小樣本壓縮率約 0.14% 到 31.7%。樣本不能外推為最終容量，但可用來決定：需完整保留的少數 log 優先壓縮，其餘只保存摘要。

## 目前狀態與文件邊界

- 盤點時沒有執行 persistent 資料、舊搬家世代或 `/var/tmp` 的刪除。
- 5TB 冷備份碟尚未連接，因此沒有執行搬移或還原測試。
- 詳細逐帳號清單保留在任務盤點文件，不複製進 shared memory。
- 個人 Drive 上的 Google 文件只是暫時閱讀副本，不是永久 canonical 紀錄；memory 不保存帳號、Google file ID 或受限文件 URL。
