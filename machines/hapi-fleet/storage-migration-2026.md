---
title: DVLab fleet persistent storage migration inventory
scope: machines/hapi-fleet
project: dvlab-storage
tags: [storage, migration, backup, cleanup, fleet]
status: active
created: 2026-08-26
updated: 2026-09-24
---

# DVLab persistent storage migration inventory

## 三批授權清理完成（2026-09-23）

- 使用者明確授權三批「驗證可移除後即移除」。已完成，共釋放 **400,975,224,832 bytes（400.98 GB）**：7 個完整資料夾及原 143 檔中的樹外 93 檔為 75,287,998,464 bytes；dvlab/backup 九帳號已驗證部分為 316,497,027,072 bytes；yochi／ML 重複檔為 9,190,199,296 bytes。這些數量是實際淨釋放，不能再當成待清候選。md1 現在 used 3,063,081,041,920、available 4,475,149,807,616 bytes，與清理前用量差額精確相符。
- 28 個差異檔已先補存至 Zeus `<admin-home>/private-system-backups/valkyrie-cleanup-delta-20260923/preserved.tar.gz`；28/28 成員集合、內容 SHA 及實際解壓讀回通過，目錄 0700、archive 0600。archive SHA `c69bc33ad1c2c30d38312c6d1080453849730efefbf8d2d6862a2a01dbaa20c6`，此小檔副本必須保留。第一批刪前 156,327 項重新檢查 identities、member-set、mount 與程序／容器引用，沿用同日已完整驗證且未變更的來源 SHA。
- dvlab/backup 九帳號完成全樹盤點；對上保存候選的一般檔重新計算完整 SHA，symlink 比較 target，並對照原組／補組冷讀回資料庫與最終 54 卷 member presence。實際移除 1,446,064 個檔案／連結；整個目錄並未清空，目前仍留 **1,704,828,928 bytes（1.70 GB）**、22,122 個檔案／連結及 5,584 個目錄。22,107 項未對上該帳號保存路徑，另外 15 個封存檔大小不同；包括快取、瀏覽器／歷史狀態與封存差異，未證明可移除者均留原處。最終 exact remaining-set 與所有保留檔 identities 檢查通過。
- 路徑不同不代表未保存：chengyin 的 `OpenSPARCT2.1.3.tar` 比其原 canonical member 大，但完整 SHA `06406bd7939c2764b81166bb4340502375ff33399193a4490eb00347d675eb1a` 對上長碟 `home/jack0716/OpenSPARC/OpenSPARCT2/OpenSPARCT2.1.3.tar`，故另驗證後移除 2,079,457,280 allocated bytes，已包含在上述九帳號總量。恢復舊 chengyin 路徑須使用回執的來源→member mapping。其他封存容器大小不同不等於資料必定遺失，但也不能直接當內容相同刪除；本輪保留 15 檔。
- yochi／部分 ML 的 29 組資料重新完整 SHA 比對，保留每組一份並刪除 65 個多餘檔；刪後與最終重查，保留檔 identities 不變，刪除路徑均不存在。ML canonical 選 Pinchun；yochi canonical 與精確 restore mapping 在回執內。不可再沿用歷史 9.2 GB 當剩餘候選。
- 現行 NFS home 未清理，只新增授權的小檔私有備份；ff945、sam031023、tyyywei 先前保留的研究 log 區與獨立 cache 清理未處理。本輪依既有冷讀回證據核對來源，沒有重新讀取外接長碟全部 payload。最初 reference checker 被既有 systemd 失效 symlink 的 ENOENT 擋下，確認 link 指向候選外缺失服務後才續行；不可忽略候選相關引用。
- 持久結案、逐檔雜湊／member 對應、壓縮回執、保留清單、現場驗證與 SHA256SUMS 在 Zeus `<admin-home>/playground/storage-audit-20260903/valkyrie-cleanup-20260923/RESULT.md` 及同目錄。原始 root 保護回執在 Valkyrie `/var/tmp/valkyrie-cleanup-20260923/`；`/var/tmp` 有 age 清理，不可只靠原始現場副本。下列本日盤點段落是執行前快照，其中「未刪／未補存」不再描述這三批現況。

## 剩餘 1.70 GB 的內容分類（2026-09-24）

- 依 09-23 封印的 `retained.jsonl.gz` 清單分類，未新增刪除、未讀取敏感檔內容。瀏覽器資料 1,281,978,368 bytes（21,221 項），主要為 SillyDuck 兩份 Chrome profile 與三份 Firefox profile；清單確有 Bookmarks、History、Cookies、Login Data、places.sqlite、logins.json 等，因此不能把整個 profile 稱為純 cache。
- pip／字型／GPU／Theano cache 與 Python bytecode 共 209,920,000 bytes（735 項）；15 個與同路徑備份大小不同的封存檔共 187,301,888 bytes，最大為 SillyDuck `boost_1_63_0.zip`，142,667,776 allocated bytes，其餘含課程作業、程式碼及舊套件封存。容器大小不同尚不足以判定內部資料不同或已完整保存。
- 其餘檔案合計 2,428,928 bytes，另有目錄配置 23,199,744 bytes。小檔含操作歷史、macOS／Windows 目錄中繼資料、SSH known_hosts、GNOME keyring／keystore，以及 SillyDuck 舊 OpenVPN 目錄的三個 `.key` 私鑰檔名（含 CA 私鑰）。這些不是全部可重建的快取；私鑰及密碼儲存狀態不可因容量小或年代舊而整批丟棄，也不可直接放入未加密公開封存。

## 整個 md1 的用量與後續盤點邊界（2026-09-23 清理前）

- 當日 live df：Valkyrie `/mnt/md1` 的 ext4 分割區 total 7,938,325,508,096、used 3,464,056,266,752、available 4,074,174,582,784 bytes（df 顯示 46%）。這是檔案系統用量，非完成的新整樹 du。全 md1 與 dvlab/backup 的兩個只讀 du 因大量小檔耗時而主動停止；不可把部分輸出當完整盤點，也沒有刪除資料。
- 下一個值得核對的非 log 區域為 `/mnt/md1/dvlab/backup`：09-04 原始 du 表為 335,848,894,464 bytes；已確認移除 ro3289/anaconda2 的 17,647,038,464 bytes，扣除後約 318.2 GB 僅是歷史估計，尚非本輪完整現況容量。LKY880154、chengyin、ro3289、hschiang、SillyDuck、yihong、music960633、kbearXD、markchang 九個重映射帳號，當日已確認都存在於最終 54 卷 readback-seen 清單，且原／補組冷讀回資料庫有對應檔案；這證明有保存來源，不證明目前 G1 整棵樹全被涵蓋，仍須內容差異核對後才可清。

## Valkyrie 非 cache 舊副本盤點（2026-09-23 清理前）

- 後續使用者要求查看更大的資料夾集合，仍只讀、未刪。15 個目標樹盤點 205,302 項；可直接列完整資料夾候選為 HugoChen `Course`、`Research` 和 arttr1521 `QFT`，合計 17,774,272,512 allocated bytes。三個 ADL21-HW1 與 qft-mapping/benchmark 另有 49,429,151,744 bytes，尚須補存 28 檔、56,379 logical bytes（15 個 Git 狀態檔、11 個 Python bytecode、2 個 QFT 產生腳本）；本輪沒有建立該補存副本。七棵樹共 67,203,424,256 bytes，與原 143 檔合併去重後為 75,287,998,464 bytes，比原批次增加 14,118,445,056 bytes，不能直接相加。
- 巢狀封存是重要保存來源：HugoChen Course.tgz／Research.tgz 的完整成員集合與旁邊展開目錄相同。146,938 一般檔全部重算 SHA、907 symlink target 與 7,751 目錄，共 155,596 項全部通過；兩個 tgz 另通過 gzip -t，且其 identity/size/mtime/ctime 與同日已對上長碟的完整 SHA 紀錄一致。僅查長碟外層 members.sqlite 會漏掉已由 tgz 保存的展開內容，不能據此斷言未備份。
- 擴大盤點的其餘十個樹完成 16,095 非目錄項目核對；同日已驗證且 metadata 未變的 139 個大檔沿用雜湊，其餘重算。最後七棵樹 156,234 項及樹外原 93 檔重新檢查，無新增、缺失或 metadata 改變；fd/cwd/exe/maps 依 inode 核對無程序引用，全部 Docker 容器無相交 mount，所有候選一般檔 nlink=1。並未重新讀取外接長碟全部 payload。
- 不可擴大至整帳號：HugoChen 其他三棵樹仍有 8,673 項、約 2.633 GB allocated 未對上本輪保存來源，多為 .o/.a/build 產物，未另外證明完整可重建；arttr1521 整個 qft-mapping/pyzx/qsyn 等也有差異。整資料夾報告、28 檔精確差異清單、壓縮證據在 Zeus `<admin-home>/playground/storage-audit-20260903/valkyrie-directory-audit-20260923/`。資料夾門檻仍需於實際刪除前重查，未補存的四棵樹不能先清。

- 使用者後續明確表示 cache 暫不處理、研究 log 風險較高先保留；改找已保存的舊資料。本輪只盤點，沒有新增刪除授權或執行刪除，現行 home 仍不動。
- 舊跨機 exact 配對 28,159 路徑重新 lstat：14,554 尚存在、13,605 已消失。不可再引用原 441.79／83.77 GiB 作剩餘可回收量。
- 第一批 143 個精確檔案通過完整來源 SHA-256 重算，並逐一對上長碟原組／補組冷讀回雜湊及最終 54 卷 member 存在性；解析 hardlink、套用最終 overlay mapping。沒有重新讀取外接碟全部 payload。候選為 HugoChen 兩個舊壓縮檔（5.097 GB）、三份 GloVe 課程資料六檔（23.469 GB）、arttr1521 的 133 個舊 QASM 輸入（30.117 GB）、jasminehsu 兩個舊安裝包（2.486 GB）。QASM 是已保存的研究輸入，不是 log，不能把整個目錄當純輸出刪除。
- 合計 logical 61,169,130,898 bytes、allocated 61,169,553,408 bytes；所有 nlink=1、realpath 未跳轉，mtime/ctime 最晚為 2023-09-05。讀取前後及最後重查來源 metadata 一致，fuser 無引用、全部 Docker 容器無覆蓋候選的 mount。實際刪除前仍須重新查使用及保存狀態。
- 精確來源→長碟 member→SHA 對應、查核結果與報告保存在 Zeus `<admin-home>/playground/storage-audit-20260903/valkyrie-duplicates-20260923/`。其他配對尚未完成本輪重驗，不列本批；此報告只允許判斷列名檔案，不支持整帳號或整樹刪除。

## 系統快取容量須扣除 Snap 硬連結（2026-09-23）

- 使用者優先尋找低風險清理項目，研究 raw log 暫留；本次只讀盤點，沒有刪除。現行 home 與受保護備份仍不在清理範圍。
- Mazu、Cthulhu、Athena、Valkyrie 的 Snap cache 一般檔全部仍有其他 hard link；單刪 `/var/lib/snapd/cache` 的檔案，預估資料區塊回收為 0。不能把 cache 的 `du` 和 disabled revision 大小直接相加。應將候選依 `(st_dev, st_ino)` 分組，只有候選名稱數涵蓋 `st_nlink` 才計入可回收區塊。
- 當次 `snap list --all` 共 48 個 disabled revisions；cache 加這些舊 revision 的聯合候選，扣除清單外 hard link 後估計 10,405,912,576 bytes。舊 revision 應由 Snap 管理器移除，保留現用 revision；代價是失去該本機舊版回退副本。五台 APT archives 另占 2,450,141,184 bytes。這些是當日估值，不是永久有效的刪除清單。
- Docker 當次顯示可回收 build cache 約 25.655 GB，但 image、container writable layer、volume 必須分開判斷；尤其不能把 unused volume 或 stopped container 當成可重建快取。本次沒有將它們列入確定可刪的資料。

## Zeus 舊備份分割區已清空（2026-09-21）

- 使用者進一步授權「確認資料在備份或新位置後即可刪除」。Zeus `<remote-home>.bak` 剩餘 33 個頂層目錄現已全部移除，釋放 975,239,655,424 bytes（975.24 GB），失敗 0；完成後剩餘項目 0、filesystem 用量 12,288 bytes、可用 3,286,104,711,168 bytes。`/dev/sda4` ext4 掛載保留，沒有格式化。現行共享 home 未清理，NIS 核心私有備份的雜湊／檔案集合／0700 與 0600 權限重查均未變。
- 16 個剩餘主要 G2 帳號依封印的 final-disposition ledger 處置。先核對 22,329 個 G2 historical payload，再套用最終 `overlay-map.jsonl`，將全部 143,954 個 sealed overlay 檔案逐項比對既有冷碟讀回資料庫的 SHA-256 與最終 54 卷成品 member 清單，全部通過。整合會將同路徑衝突移到 `.archive/conflicts/canonical-overlay-20260905/`；不能直接用舊 overlay 路徑判成備份缺失，必須套用 authoritative mapping。主要帳號的 current-exact、可重建環境及敏感狀態仍依既定整合規則處置，不代表所有原始 bytes 都在長碟。
- 13 個小帳號與 3 個整理工作目錄另外完整保存到 Mazu `/usr/2TB-SSD/backup-work/zeus-homebak-retained-20260921/preserved.tar.zst`，134,911,736,531 bytes；SHA-256 `73bf96ddfc2b1d6d83a36242c4981ddf0e061f6b02bb2f438f7b7518fcd4a034`。完整 zstd 解壓、GNU tar 來源內容／metadata 比對與 exact member-set 驗證均通過，共 755,239 個項目、零差異。目錄 0700、archive 0600，保留原 numeric owner、mode、mtime、symlink/hardlink 及 tar ACL/xattr；舊帳號私有狀態可能在內，不應直接加入未加密外接 Home archive。
- 這份 SSD 封存檔是受保護的保存副本，不能因父目錄叫 `backup-work` 就當可重建暫存刪掉。三個原 Zeus 工作目錄（含 `.work`）與 13 個小帳號的整棵原始內容均由此副本保存；後續查找舊整理證據應到此檔或維運 audit，不能再假設 Zeus 舊路徑存在。恢復時先解到隔離位置，核對帳號名與歷史 numeric ownership。
- 刪除前核對 exact root identity、ext4 mount、無巢狀掛載及 process/container/config references 為 0；主要 G2 檔案在整合後沒有新 mtime/ctime，例外僅已知 2023 ctime 的 Bazel 未來 mtime。33 個 root 回執與空目錄 live check 全部通過。持久證據在 `<admin-home>/playground/storage-audit-20260903/homebak-final-cleanup-20260921/RESULT.md` 與同目錄 JSON、SHA、logs；原始現場紀錄在 Zeus `/var/tmp/homebak-final-cleanup-20260921/`。下方較早的 NIS 批次與分割區盤點為移除前歷史，不再表示整個舊分割區仍有資料。

## 2026-09-21 本機舊資料清理結案

- 使用者在檢閱 248 項精確清單後明確授權移除，現已全部完成，失敗 0、跳過 0。Zeus 94 項釋放 511,352,344,576 bytes；Valkyrie 153 項合計釋放 593,244,368,896 bytes；Cthulhu 單一孤立 buildx volume 釋放 15,747,387,392 bytes，三台磁碟淨用量共下降 1,120,344,100,864 bytes（約 1.12 TB）。系統碟背景活動可能使 df 差額與原估值有微小差異。
- 刪除前 248 項重新逐樹核對檔案數、容量、mtime/ctime、filesystem 與程序 inode 引用，均符合前次盤點；各主機執行前重查程序／容器引用，逐項核對 root inode、時間、實際路徑及 mount。只刪精確核准的目錄；Cthulhu 使用 `docker volume rm buildx_buildkit_qsyn-builder0_state`，未做廣泛 prune。完成後 248 路徑均確認不存在、Docker volume 已消失，23 個同機保留路徑仍存在，Zeus 歷史整合 staging 保留。現行 NFS home 未清理；Mazu、Athena、外接碟及 scratch 均不在此批範圍。
- 完成紀錄在 Zeus `/var/tmp/old-data-audit-20260921/removal-20260921/RESULT.md` 與 `complete.json`，各主機另保留 root 保護的 `/var/tmp/old-data-removal-20260921-<host>/receipt.jsonl`，遠端／本機回執 SHA 已核對一致。原盤點報告與原校驗清單已封存；不能再假設 09-05 exact-duplicate 對照表指定的 Zeus canonical 路徑全部仍存在，後續清理須重新核對保存來源。

- 清理候選必須逐子樹核對，不能只看頂層目錄時間。這次使用完整子樹的最新 mtime 與 ctime 均超過 90 天、既有備份整合或可重建證據、程序 inode/path 引用及容器 mount 檢查；不使用會被盤點讀取影響的 atime 判定歷史閒置。
- Zeus 多數主要 G2 帳號的 `.git` 目錄在 2026-09-05 有 metadata 更新；即使帳號根目錄顯示 2024，也不能直接宣稱整個帳號多年未動。帳號整體與其中獨立環境/cache 必須分開判定，且容量不可重複加總。
- `jasminehsu/anaconda3` 的 Bazel `A-server.jar` 帶 2032 年 mtime，但 ctime 為 2023 年；這是時間資料不一致，不能把未來 mtime 直接描述為近期使用，也不能未釐清就通過嚴格閒置篩選。
- 273 個候選逐樹檢查完成，248 個不重疊項目通過：Zeus 一個完整舊帳號加 93 個環境/cache、Valkyrie 122 個 G1 環境/cache 加 31 個隱藏本地 home 環境/cache、Cthulhu 一個孤立 buildx volume。扣除外部硬連結後估計 1,120,344,932,352 bytes；這是移除前候選估值，後續已依授權完成，上方為實際結案結果。詳細現場證據、`REPORT.md` 與 `eligible-paths.tsv` 留在 Zeus `/var/tmp/old-data-audit-20260921/`，是有日期的盤點而非永久刪除授權。現行 NFS home、其內 G3/G4 archive、scratch、現役服務與未證明可重建的工作資料不在本次清理候選範圍。實際移除前重查引用及保存副本，清單本身不證明未來仍無使用。

## NIS 核心另存現行 home 完成（2026-09-21）

- 使用者授權保留 NIS 核心於現行 home 並刪除其餘內容。已保存至 Zeus 管理帳號的 `<admin-home>/private-system-backups/nis-core-20260921/`；目錄 0700、六個備份／metadata 檔案均為管理帳號持有且 0600，其他一般使用者不可讀。`nis-core.tar.gz` 167,169 bytes，保存 59 個一般檔、8 目錄、1 symlink；含原 numeric owner/mode/time 與 tar 可保存的 ACL/xattr。`source-manifest.json`、`SHA256SUMS`、`VERIFIED.json`、`README.md` 與 `RELOCATION-COMPLETE.json` 同目錄保留。這是含舊密碼雜湊／NIS maps 的私有系統備份，不是可納入一般未加密 Home 成品的新 payload。
- 完整逐檔 SHA-256、archive 成員集合、owner/group/mode metadata、實際隔離解壓後內容／檔案 mtime／symlink 比對均通過；原始絕對 `securenets` link 保留，未跟隨去複製現行設定。移除前再次核對來源 inode/mtime/ctime/hash、backup SHA、引用、ext4 mount 及無巢狀掛載，才刪除兩個精確來源 `nis_transfer` 與 `backup-nis-2604`（包含不保留的 shell 設定／歷史／補完資料），釋放 11,849,728 bytes。最後確認兩目錄不存在、私有備份仍完整；root 回執在 Zeus `/var/tmp/nis-core-relocation-complete-20260921.json`。
- NIS 批次當時只處理兩個來源，尚留約 975 GB 舊帳號與整理工作目錄；之後使用者另行授權整個舊分割區清空，已依上方「Zeus 舊備份分割區已清空」完成。下方兩個 NIS 路徑仍存在的敘述均為移除前盤點。

## Zeus 整個舊備份分割區的移除前盤點（2026-09-21）

- 第一批移除後，`<remote-home>.bak` 仍是 `/dev/sda4` 的獨立 ext4 掛載，用量 975,251,517,440 bytes；不等於現行 NFS home。
- 頂層另含兩份系統帳號／NIS 備份：`nis_transfer`（11,268,096 allocated bytes，含 `yp` 與 `sysfiles`）、`backup-nis-2604`（581,632 bytes，含 passwd/group/shadow/yp）。兩份合計約 11.85 MB；檢查只讀名稱與 metadata，未輸出敏感檔案內容。兩者未列入已讀取的 single-home 帳號計畫或 G2 最終帳本，不能由 Canonical Home 完成標記推論已覆蓋，整碟清空前須另外保存或確認去向。
- 後續內容確認：`nis_transfer/sysfiles` 是 2023 年遷移用 passwd/group/shadow/gshadow 匯出，`nis_transfer/yp` 是 NIS maps、Makefile 與 binding；`backup-nis-2604` 為 2026-08-22 的 passwd/group/shadow/yp 快照。三個核心範圍共 1,025,578 logical bytes（一般檔占用約 1.09 MB），其餘主要為舊 shell 設定、歷史與補完資料。適合另存為受限權限的系統歷史備份，不必因此保留整個舊備份分割區。
- 僅在主機內比較並輸出統計：2023 的 51 帳號與 2026 的 150 帳號皆存在現行 `/etc/passwd`，UID/GID 全同；兩代群組也無缺漏或 GID 改變，但各有 1 筆成員不同。舊 shadow 的密碼欄位與現行分別有 4／1 筆不同，未輸出或保存任何 hash／欄位值。因此沒有發現尚未搬入現行的帳號 identity，主要保留價值是歷史認證／權限狀態，不是唯一研究資料；不可把舊快照當成現行設定。
- `backup-nis-2604/yp/securenets` 是指向 `/etc/ypserv.securenets` 的絕對 symlink，並非獨立保存的舊設定內容；封存／還原時須區分連結與歷史檔案本體。含 shadow 與 NIS maps 的備份按系統憑證資料保護，不應因體積小就併入一般未加密 Home 備份。
- 三個備份工作目錄 `single-home-build-20260905`、`canonical-home-one-touch-20260906`、`nfs-canonical-20260905` 合計 164,573,159,424 allocated bytes，含 manifests、差異整合 payload 與 `.work`；不能把它們當成舊使用者帳號。其餘主要為約 810.67 GB 舊帳號資料。90 天閒置條件只用於第一批篩選，9 月備份整理更新過 metadata 不代表永久不可清；是否可整批清除應依保存內容覆蓋與工作紀錄去向核對。此輪只有檢查，沒有擴大刪除。

## 2026-09-21 冷備份位置更新

Jonathan 唯一 canonical 已完整搬至長備份 One Touch（UUID `001D-7DC1`）的 `backup/home/jonathan.tar.zst` 與 SHA sidecar；目的檔 76,204,875,019 bytes，完整 SHA-256 與既有 `c5fc01e0405696dd75468d80a7d84ea5630890b1c4a2b19a3659c0df036d9cdf` 相符，解壓及 257,533-member tar 讀回通過後才刪短碟來源。兩顆短備份碟均已依使用者授權清空；4 TB 短碟的全量內容比對在 10/21 檔通過後按要求停止，勿宣稱全量比對完成。下方 09-04／09-06 的 Ultra Touch 保存位置及來源保留狀態為歷史紀錄；現況與證據見 [外接備份碟](../mazu/external-backup-drives.md)。Jonathan 仍為永久保護項，且獨立於 Canonical Home 54 卷之外。

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
- 冷備份 session 的 current-home 處理分母是 69，因為現行 70 accounts 中扣除已單獨封存的 Jonathan；最終單一 home 的 71 identities = 69 個一般 current homes + Jonathan protected canonical + `b10901099` legacy-only canonical。這是帳號邊界，不是備份進度百分比。
- 69 個 current homes 的封存前 inventory 共 23,755,641 個 directory／regular file／symlink 節點。依既定規則排除 5,116 個可重建根目錄、2,548,557 個節點後，current input 為 21,207,084 個節點；根目錄分類是 env 73、EDA output 4,877、Rust build 48、package cache 26、其他 build output 92。另保存 6,742 筆套件重建線索，metadata error 為 0。這些數字是 manifest 建構結果，不是已完成冷備份的進度。
- current manifests 位於 Zeus `<single-home-build-root>/current-inputs.unfiltered.nul` 與 `current-inputs.nul`；同目錄另有 `overlay-inputs.nul`、`conflict-inputs.nul`、`overlay-map.tsv`。current 掃描採 `find -xdev` 且只納入 directory、regular file、symlink；sealed overlay 最終加入 161,274 個輸入節點，其中 143,954 個 regular files 已由既有 SHA manifest 完整讀回。衝突內容必須放到 account 內的 generation archive 路徑，不能覆寫 current 同名檔。
- sealed overlay 與逐帳號 fragments 只是「舊世代相對 current 的必要差異」，不是 standalone home。不得單獨封裝後冒充 canonical `copy.tgz`：它們沒有 current base，且 `b09901037`、`b10901098` 因與 current 完全相同而刻意無 fragment。如需單獨保存 delta archive，必須標明 non-standalone 並保留 account/disposition manifests；完整 canonical 仍必須合併 current base + overlay 成單一 `home/<account>/`。
- G2 staging 曾把 current 已有的 dirty/untracked 檔重複納入；root cause 是 staging 缺少 same-relative-path SHA filter。共 26,069 個 current-exact 檔、2,648,075,407 bytes 與 162 個 exact symlink 已在每次 `cmp`/hash 後移到 sibling `.work/g2-current-exact-hold-*`，hold manifest/readback 全通過。`.work` 是復原與證據，不是 cold-backup payload。
- 不可在 `.git/` 內做 content hardlink：Git refs 可能被錯誤連結而互相改寫。跨 home 精確重複可在最終 archive payload 層處理，但 Git object/ref 結構維持獨立。
- 未加密 cold archive 明確排除 SSH/GPG identity、credentials/token、browser/history/Xauthority 及 editor/cache/runtime。current+overlay 必須先寫成單一 `home/<account>/`、做 archive test、逐檔 SHA-256 與冷碟完整讀回；在人工授權前，G2/G3/G4 來源都只能是 delete-later，不能實際刪除。
- damaged short-drive `copy.tgz` 最終 CRC 不合格，不能當 canonical input，也不應重建 archive-of-archives；健康 NFS current 與 sealed overlay 才是單一-home 冷備份輸入。本 NFS session 沒有 mount、讀寫或清理兩顆外接碟，寫入權已正式交接給外接碟 session。
- 冷備份串流使用 GNU tar POSIX format、numeric owner、sparse、no-recursion、NUL/verbatim file lists，依序合入 current、overlay、conflict manifests，再由 `zstd -T4 -5 --long=27` 壓縮，遠端以 `dd bs=16M conv=fsync` 寫入 `.partial`。對應的 build、readback、validation、publish scripts 保存在 `<admin-home>/short-backup-handoff/`；publish 必須等 zstd test、完整 member/file manifest、SHA-256 與冷碟 full readback 全通過。
- 2026-09-06 的首次串流依使用者要求停止；`<long-backup-mount>/backup/home/single-home.tar.zst.partial` 停在 405,723,078,656 bytes。當時 tar stderr 記錄 28 個掃描後消失的 live NFS paths，但工作未完成，且沒有 archive SHA、zstd test、完整 member manifest 或 cold readback。因此該 partial 明確不是 canonical、不可 publish 或取代任何來源；相關 HAPI jobs 與本地／遠端 writer 已清為 0，除非有新的明確指示不得自行續跑。

- SHA handoff manifests 使用相對路徑；`sha256sum -c` 必須從 manifest 指定的 audit/root 目錄執行。從其他 working directory 執行會產生假性 missing/FAILED，不能據此判定 payload 壞掉。

## 目前狀態與文件邊界

- 除上列 Jonathan 已完成批次外，其餘 persistent 資料與 G2/G3/G4 舊搬家世代未因本次工作刪除；NFS 合併只新增 Zeus 本地資料碟上的 sealed canonical overlay 與 sibling evidence/holds。
- 2026-09-04 Mazu 同時掛載兩顆冷備份碟；Jonathan 寫入只短暫將 Ultra Touch remount RW，完成 sync 與三輪完整讀驗證後已回到唯讀。One Touch 未改寫。
- 長備份碟的 5 個外層 tgz、4,853,391 筆原始 member、4,096,391 筆巢狀 tar member 與 600,910 筆其他巢狀 archive member 均查無 `Jonathan` 路徑，以及 `mix1.tar`、`mix2.tar`、`mix3.tar`、`1min.tar`、`pack.tar`。`jon*` 命中只有圖片、Java 類名、zsh theme 與 IsolatedStorage 隨機目錄，不能誤認為 Jonathan home。
- 短備份碟外層有 45 個 tgz，沒有 `jon*` 檔名；其中 G4 `copy.tgz` 對應 NAS 仍存在的 `copy/cthulhu_home`。該來源目錄精確是 19 個其他帳號 tgz、搬移腳本與帳號清單，沒有 Jonathan；21 個檔案加兩層目錄的預期 tar 長度 `279240816640` bytes，和 `copy.tgz` gzip trailer ISIZE 同為 `67942400`（模 `2^32`）。2026-09-04 全串流外層 member 掃描直到 gzip trailer 才以 CRC error 結束，未命中任何 `jon*.tgz` 或五個 Jonathan 目標 tar；因此外層名稱可否定，但 `copy.tgz` payload 完整性不合格，也不能據此否定內容藏在其他帳號巢狀 tgz 內。不可把它當成健康備份或直接刪除。
- 詳細逐帳號清單保留在任務盤點文件，不複製進 shared memory。
- 個人 Drive 上的 Google 文件只是暫時閱讀副本，不是永久 canonical 紀錄；memory 不保存帳號、Google file ID 或受限文件 URL。
