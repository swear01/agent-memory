---
title: DVLab fleet persistent storage migration inventory
scope: machines/hapi-fleet
project: dvlab-storage
tags: [storage, migration, backup, cleanup, fleet]
status: active
created: 2026-08-26
updated: 2026-09-05
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
- 四個冷帳號已定位 3.749 TiB（67.39%）的批次 log、proof trace、testcase output 與 tar 內精確重複。這一層不是整個帳號直接刪除：先抽出 source、論文、重現腳本、final/best 結果與摘要，再刪或壓縮原始輸出。特別是 tyyywei 的待精簡樹內混有 19,051 個 QDIMACS/AIG input（36.247 GiB），不能整個目錄移除。
- 最大帳號有四個未壓縮 tar，合計 1.700 TiB；完整 member inventory 共 3,686 個 regular files，全部為 `.log`、沒有 non-log。相鄰的 `pack.tar` 則有 316.890 GiB 一般檔案內容，其中 `.log` 只佔 98.137 GiB（30.97%）；其餘主要是 SAT benchmark 125.072 GiB、proof 28.609 GiB、舊工具安裝包 31.503 GiB 與 WebProg 13.174 GiB，必須分開判讀。
- `pack.tar` 是 Jonathan 人工整理的舊 home/workspace 搬家封存：member mtime 為 1996–2019、tar 本身 mtime 為 2023，根目錄與 shell alias 都對應 `workspace/sat`、`workspace/RPgen` 等工作路徑；不是套件 cache 或純 log archive。
- `pack.tar` 的 SAT raw log 有相鄰執行設定、特徵萃取 scripts、case/status/runtime 與 train/test/validation CSV 摘要，適合在保留 manifest、source/config、摘要及少量異常案例後精簡。62,223 個 CNF 經完整 SHA-256 後有 820 組、1,390 個多餘 member，共 44.469 GiB 精確重複；保留每組一份可將 CNF 從 125.072 GiB 降到 80.603 GiB。排除 98.134 GiB raw log 與這批 exact CNF 後，第一版 slim tar 的一般檔案內容約 174.287 GiB。`.proof`/RUP certificate、舊版授權工具與非 log 專案不能沿用同一刪除判斷。
- Jonathan 四個 pure-log tar 的 method／timeout／summary frame 已完成對齊：`mix1/2/3` 的 1,013 個 mix case 全有完整 30 秒 frame，`1min.tar` 的 330 個 mix 與 1,343 個 rup case 全有完整 60 秒 frame。1,000 個無法可靠分類的 3lit log 全留，另對五個 cohort 各留大小極值；外層共留 1,010 檔、11,726,501,020 bytes，排除 2,676 檔、1,857,516,134,377 bytes。
- `du` 與表觀檔案大小都要保留：某個訓練 `tmp` 目錄表觀約 87 GiB、實際配置僅 12.93 GiB，因 sparse files 不能把 `st_size` 當成回收容量。
- G1 內部大於等於 1 GiB 的同大小候選，尺寸法推測 79.41 GiB 重複；完整 SHA-256 後只剩 16 組、43.21 GiB。再擴大掃描 yochi 專案後，扣除既有 exact/rebuildable 與大型輸出區的新增精確重複更新為 8.559 GiB，主要是生成 output、課程 dataset，以及跨帳號 ML dataset/archive；仍須先指定 canonical owner。另有同檔名同大小但雜湊不同的 benchmark input，明確不可刪。
- 巨型文字輸出的 zstd level-1 小樣本壓縮率約 0.14% 到 31.7%。樣本不能外推為最終容量，但可用來決定：需完整保留的少數 log 優先壓縮，其餘只保存摘要。

## Jonathan canonical 完成狀態（2026-09-04）

- 將現行 NFS 非敏感設定、legacy 非敏感設定、精簡 `pack.tar` 與必要 raw-log 樣本合併為單一 `jonathan/`。現行 NFS `<remote-home>/jonathan` 保留原地；SSH/GPG identity、`.netrc`、browser、history、`.Xauthority`、cache/runtime 不寫入未加密外接碟。
- `pack.tar` 排除 105,370,504,056-byte SAT raw-log tree、472,182 個 `node_modules`、bytecode、7,923,939,835-byte waveform、5,139,276,539-byte RPgen `patch.v` 與三個外部 CAD symlink；保留 source/Git/摘要/proof/唯一 benchmark/舊工具安裝包。另抽回 `pack.tar` 兩個代表 log；全體樣本共 1,012 檔、12,173,965,999 bytes。
- SAT benchmark 全部路徑保留；`hardlink --content` 對 62,223 檔正式連結 1,385 個精確重複檔，節省 44.47 GiB。canonical staging 加入樣本後實佔 184,335,990,784 bytes。
- 唯一長期備份是 Mazu Ultra Touch mountpoint `/mnt/ultra-touch` 下的 `backup`/`home`/`jonathan.tar.zst`：76,204,875,019 bytes，SHA-256 `c5fc01e0405696dd75468d80a7d84ea5630890b1c4a2b19a3659c0df036d9cdf`。`zstd -tq`、257,533-member 完整 tar list、內含 160,099-file SHA manifest 與 README 讀回 `cmp` 均通過；碟已回到唯讀。此檔是 Jonathan 唯一 canonical home，屬永久保護項，不得列入 duplicate/delete candidates；只有另一份 replacement 通過逐 byte、SHA-256 與 archive 讀回驗證並取得人工授權後，才可替換或移除。
- 刪除前確認來源為 Valkyrie 本地 ext4 且 `proc_refs=0`。`/mnt/md1/jonathan` 已刪除，`df` 實測釋放 2,211,264,499,712 bytes；現行 NFS home 未動。Mazu staging 也已移除並釋放 184,431,239,168 bytes。恢復只能依上述 canonical archive 與 SHA-256。

## NFS 單一 home 合併（2026-09-05）

- `192.168.1.199` 與 `192.168.1.200` 是同一台 NAS、同一份 `/volume1/nfs-home`，不能當成兩份備份。現行 NFS 70 個帳號共 10,047,759,310,516 logical bytes；NFS 內 G3 `.backup/*.tar` 1,313,249,259,520 bytes、G4 `copy/cthulhu_home/*.tgz` 279,240,795,714 bytes。
- 最終格式只能有一個 `home/` 與每帳號一個 `home/<account>/`。current 是現役帳號 base，舊世代 unique 內容以 `.archive/<generation>/` overlay；同路徑不同內容不可覆寫。帳號名是 canonical identity，不能只看 numeric UID/GID：舊 `b09901005` 的 1041 現已解析成 `ice890425`，restore 前要人工決定 owner。
- G4 19/19、G3 29/29、主要 G2 17/17 已全部定性與收斂。原始來源分別是 279,240,795,714、1,313,249,259,520、1,274,891,141,120 bytes，合計 2,867,381,196,354 bytes（2.6079 TiB）；來源仍原地保護，沒有在本階段刪除。
- 必要差異已封印在 Zeus `<remote-home>.bak/nfs-canonical-20260905/home`：26 accounts、143,954 regular files、1,965 symlinks、36,046,476,712 bytes（33.57 GiB）。其中 G4/G3/G2 payload 分別為 12,556,927,871、1,668,027,537、21,821,521,304 bytes；全檔 manifest 自身 SHA-256 是 `d4c61f00aa4719356cc9002f4245d76cc7aff6e9da73ec85e0907cc1c763eed2`，143,954/143,954 讀回通過，symlink escape、forbidden directory、high-confidence secret hit 均為 0。
- 現行 70 accounts 與 overlay 的聯集是 71 accounts，唯一新增 alumni identity 為 `b10901099`。Jonathan 不在這批 overlay；既有 Ultra Touch `backup/home/jonathan.tar.zst` 仍是唯一永久保護 canonical，不得另建第二份。
- G2 staging 曾把 current 已有的 dirty/untracked 檔重複納入；root cause 是 staging 缺少 same-relative-path SHA filter。共 26,069 個 current-exact 檔、2,648,075,407 bytes 與 162 個 exact symlink 已在每次 `cmp`/hash 後移到 sibling `.work/g2-current-exact-hold-*`，hold manifest/readback 全通過。`.work` 是復原與證據，不是 cold-backup payload。
- 不可在 `.git/` 內做 content hardlink：Git refs 可能被錯誤連結而互相改寫。跨 home 精確重複可在最終 archive payload 層處理，但 Git object/ref 結構維持獨立。
- 未加密 cold archive 明確排除 SSH/GPG identity、credentials/token、browser/history/Xauthority 及 editor/cache/runtime。current+overlay 必須先寫成單一 `home/<account>/`、做 archive test、逐檔 SHA-256 與冷碟完整讀回；在人工授權前，G2/G3/G4 來源都只能是 delete-later，不能實際刪除。
- damaged short-drive `copy.tgz` 最終 CRC 不合格，不能當 canonical input，也不應重建 archive-of-archives；健康 NFS current 與 sealed overlay 才是單一-home 冷備份輸入。本 NFS session 沒有 mount、讀寫或清理兩顆外接碟，寫入權已正式交接給外接碟 session。

## 目前狀態與文件邊界

- 除上列 Jonathan 已完成批次外，其餘 persistent 資料與 G2/G3/G4 舊搬家世代未因本次工作刪除；NFS 合併只新增 Zeus 本地資料碟上的 sealed canonical overlay 與 sibling evidence/holds。
- 2026-09-04 Mazu 同時掛載兩顆冷備份碟；Jonathan 寫入只短暫將 Ultra Touch remount RW，完成 sync 與三輪完整讀驗證後已回到唯讀。One Touch 未改寫。
- 長備份碟的 5 個外層 tgz、4,853,391 筆原始 member、4,096,391 筆巢狀 tar member 與 600,910 筆其他巢狀 archive member 均查無 `Jonathan` 路徑，以及 `mix1.tar`、`mix2.tar`、`mix3.tar`、`1min.tar`、`pack.tar`。`jon*` 命中只有圖片、Java 類名、zsh theme 與 IsolatedStorage 隨機目錄，不能誤認為 Jonathan home。
- 短備份碟外層有 45 個 tgz，沒有 `jon*` 檔名；其中 G4 `copy.tgz` 對應 NAS 仍存在的 `copy/cthulhu_home`。該來源目錄精確是 19 個其他帳號 tgz、搬移腳本與帳號清單，沒有 Jonathan；21 個檔案加兩層目錄的預期 tar 長度 `279240816640` bytes，和 `copy.tgz` gzip trailer ISIZE 同為 `67942400`（模 `2^32`）。2026-09-04 全串流外層 member 掃描直到 gzip trailer 才以 CRC error 結束，未命中任何 `jon*.tgz` 或五個 Jonathan 目標 tar；因此外層名稱可否定，但 `copy.tgz` payload 完整性不合格，也不能據此否定內容藏在其他帳號巢狀 tgz 內。不可把它當成健康備份或直接刪除。
- 詳細逐帳號清單保留在任務盤點文件，不複製進 shared memory。
- 個人 Drive 上的 Google 文件只是暫時閱讀副本，不是永久 canonical 紀錄；memory 不保存帳號、Google file ID 或受限文件 URL。
