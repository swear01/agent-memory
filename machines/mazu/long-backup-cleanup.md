---
title: Mazu 長備份可重建資料清理邊界
scope: machines/mazu
machine: mazu
tags: [backup, archive, venv, conda, git, cmake, deduplication]
status: active
created: 2026-09-04
updated: 2026-09-12
---

# Mazu 長備份可重建資料清理邊界

## 使用者原則

- 2026-09-12 接續執行決策（取代先前僅評估狀態）：使用者已同意「保留原卷＋另做補充」並要求高速設定。工作根目錄為 Mazu `/usr/2TB-SSD/backup-work/canonical-recovery-20260912`；原 48 卷和原 staging 不刪改。`canonical-recovery-readback-20260912.service` 正式讀回，通過 decoder exit/decoded length、逐檔 SHA 與原始已提交 metadata 比對後，以 `OnSuccess=canonical-recovery-supplement-20260912.service` 自動銜接 planner、壓縮、冷讀回、union coverage／歷史 required hashes、分卷 SHA 與還原 metadata。使用者隨後澄清高速必須保留原本壓縮效率；在補壓尚未啟動時已修正為 `-mx=5 -m0=lzma2:d=512m -mmt=16 -v32g`，保留原一般等級與 512 MiB 字典，只把執行緒由 4 增至 16。不得自行把高速解讀成降低等級／字典；未宣稱相同壓縮率或實際大資料加速倍率。修正參數的實際 encode/readback/restore 測試通過，原卷讀回未中斷。依實際 node type 排除 FIFO/socket/char/block device；連結和已選定的 aigfuzz 保留。跨組歷史硬連結記錄 inode 對照並核對內容 SHA；還原 helper 先驗 SHA 再接回硬連結，最後恢復目錄 mtime。新資料庫使用 Python tarfile `stream=True`，避免全體 TarInfo 常駐記憶體。測試通過完整／截斷串流、FIFO、損毀 header、原始 SHA mismatch、高速 7z encode/readback、union、實際還原和跨組硬連結。這是啟動驗證，**不是正式資料已驗證完成**；唯一新完成閘門是工作目錄 `complete.json`，原 runner success/publication markers 不沿用。新補充名稱為 `Canonical-Home.supplement-20260912`，驗證後 metadata 目錄為 `Canonical-Home.recovery-20260912`；任一步失敗保留 evidence，不自動重跑。systemd units 位於 `/run`，重開機不保證自動接續；初始小段讀回為補上完整 tar prefix 邊界而重啟，舊分析目錄保留為 `readback-attempt-before-prefix-checkpoint`。Mazu 該 NFS mount 的臨時 readahead 已由 1024 恢復為原值 128 KiB。補充 service 的 `ExecStopPost` 會核對常備碟 UUID 再 sync/remount-ro。此後查狀態應查新 units 與 SSD `original/progress.json`／`progress.json`／`failure.json`，不能再只看舊壓縮 PID。

- 2026-09-12 查證：原串流於 09-11 19:56 結束，pipeline status 為 tar=141、tee=141、7zz=0、validator=1。validator 拒收歷史 staging 中 `.steam/steam.pipe` FIFO（tar type `6`），不是來源消失錯誤；來源消失使用者於 09-11 已接受不再追查。48 卷共 1,636,490,700,048 bytes，7zz l 能讀取 headers，內含 tar 5,392,407,028,833 bytes，但尚未做完整讀回，不能宣稱 payload 完整。FIFO 位於 history-inputs 第 822,936 筆，current-inputs 不含此路徑；runner 順序表明已走完 current 階段才在 history 中斷。SQLite 最後已提交 rowid=22,220,000；每 10,000 筆提交且 tee 有緩衝，DB 不是精確封存終點。初次調查提出保留原卷、完整串流讀回建立有效成員清單後另建補充 archive；其後使用者已授權，執行狀態見本節 2026-09-12 接續執行決策。特殊節點應按實際類型排除 FIFO/socket/device，保留一般檔案、目錄與連結，不靠副檔名判定。

- 2026-09-06 Canonical Home 分卷要求：一個邏輯壓縮檔切成多個實體 volume，
  不是按帳號／子樹各建獨立 archive。常備碟的 current、歷史 overlay 與原有檔案
  納入同一邏輯 Home；短備份碟暫不讀寫。字典大小、實體卷大小、UTF-8 編碼
  是三個不同概念，不可混淆。
- 長備份只保留不可替代資料；核心程式碼必須保留。
- Python／Conda env 改留可重建 dossier，環境本體可在 dossier 驗證後排除。
- 可復現的編譯輸出可排除；大型 log、dataset、checkpoint、影音由使用者判斷。
- Git 專案可壓縮保留；歷史相較 worktree 異常龐大時另行決定，不在原始備份執行
  `gc`、`repack` 或 `prune`。

## 已驗證範圍與數值

分析使用既有的 9,550,692 筆 archive member inventory 與 1,240/1,240
payload SHA-256 結果，未重新讀取外接碟。正確 JSON 欄位是
`venv.real_environment_roots=98`；原始 476 筆候選包含 378 筆誤判。

98 個真環境合計約 72.0 GiB logical，外層 archive 估計約 47.6 GiB：

- 50 個：已有 Python 版本與 package metadata；完成 dossier 後可排除環境本體，
  約 31.3 GiB logical／17.7 GiB archive heuristic。
- 33 個：含 `.egg-link`、`direct_url.json` 或 private/editable manifest；必須先證明
  被引用的原始碼另有保留，約 24.3／15.3 GiB。
- 14 個：位於已知尾端截斷的 archive，維持 HOLD，約 12.4 GiB logical。
- 1 個：只見 Python 版本線索，需小型 metadata 才能形成 dossier。

上列 `50/33/14/1` 是讀取小型 metadata 前的初步分組，不是最終排除決策。
2026-09-04 完成 11,153/11,153 個小型 metadata 讀取後，98 份 dossier 的保守決策為：

- 3 個 `EXCLUDE_AFTER_DOSSIER`：約 1.03 GiB logical／0.49 GiB archive heuristic。
- 95 個 `HOLD`：約 67.0 GiB logical／44.9 GiB archive heuristic。
- 94 個環境仍有未完成的 local/editable project source mapping；75 個已找到相鄰的
  project manifest，19 個尚未找到。這不代表環境是假的，而是排除整棵環境前仍要證明
  被引用的原始碼已另行保留。
- 46 個 HOLD 只有 source mapping 這一項阻礙，約 17.2 GiB logical／10.3 GiB archive
  heuristic；其中 40 個已有 linked manifest，是下一批最容易解除 HOLD 的目標。
- 36 個環境根目錄混有非標準檔案（可能包含 editable source），不可整棵排除；14 個位於
  inventory 不完整的截斷 container。這些集合彼此重疊。

`local/editable project source mapping` 是暫時性的刪除閘門，不代表 dependency 本身不能
重建。只要確認被 `.egg-link`、`.pth`、`direct_url.json` 或 Pipenv `.project` 指向的
source tree 已保留，並保存 package/version/platform、manifest/lockfile，以及適用時的
VCS URL 與 commit，就能把環境本體改判為可排除。只有 package name 不足以覆蓋未發布的
editable/private code。94 個 mapping HOLD 中已有 75 個找到相鄰 project manifest；46 個
只有 mapping 這一項 blocker（其中 40 個已有 linked manifest），應優先核對而非永久保留。

先前的 33 個是小型 metadata pass 前，以 local/editable marker 粗分出的 provisional
category（24 個 `.egg-link`、6 個 private/editable manifest、2 個 `direct_url.json`、
1 個 private manifest）；不可把它當成最後的不可刪數量。

## `cph.tgz` 完整性

巢狀檔 `yoctol.tgz!yoctol/cph.tgz` 的外層 inventory 宣告大小與實際抽出的 payload 大小
都是 37,711,249,408 bytes（35.121 GiB），所以不是稽核抽取時少複製；損壞位於既存的
`cph.tgz` 本身。`gzip -t` 回報 unexpected end of file，GNU tar 回報 unexpected EOF，
bsdtar 在讀取 archive-relative path
`home/cph/LibriSpeech/LibriSpeech/train-clean-100/1737/142397/1737-142397-0000-norm.wav`
時回報 truncated input。

截斷前仍可列出 283,594 個 members（39,040 directories、243,815 files、739 other），
已宣告的 readable file bytes 約 78.291 GiB logical。2026-09-04 重新從唯讀外接碟抽取後，
payload 長度仍是 37,711,249,408 bytes，SHA-256 仍是
`fd4244733ec1faa81884ae5859347ac900dcf34ed16b466f788ae2ab4098a841`；kernel journal 沒有
USB、block I/O 或 exFAT read error，證明這是穩定存在的 archive truncation，不是此次讀取
失敗或檔案持續劣化。

前 283,593 個已列出的 members 可順序讀完；第 283,594 個也是最後一個可見 header，亦即
上述 WAV。它宣告 467,918 bytes，實際可抽出 356,352 bytes，缺 111,566 bytes。可抽出的
部分是有效 Microsoft PCM（16-bit、mono、16 kHz）：header 宣告 233,920 frames／14.620 s，
實際可解碼 178,137 frames／11.134 s，最後 55,783 frames／3.486 s 缺失。舊 bsdtar 訊息
`needed 467968 bytes, only 0 available` 是 skip/padding 路徑的描述，不能當成實際可救資料量。

因此「現存且有 header 的成員」只有最後這一個不完整，但 archive 沒有正常 tar/gzip 結尾；
原本是否還計畫寫入其他、尚未產生 header 的 members，無法由截斷檔本身回答。現有證據也
無法區分是原始建立中斷、複製中斷，或來源檔本來就壞。

Trash 中 57,183,305,728-byte 的舊 `yoctol.2.tgz` 只列出 `yoctol/`、完整的
`achiang.tgz` header 與同樣宣告 37,711,249,408 bytes 的 `cph.tgz` header，之後外層即
unexpected EOF，沒有第四個 member；它只是更短的 `cph.tgz` 前綴，不能補回位於尾端的 WAV。

此 WAV 的前 11.134 s 可直接 salvage；遺失的 3.486 s 不能由現有 compressed bytes 推算。
但 archive 另保留 `home/cph/voicefilter/utils/normalize-resample.sh`，命令是
`ffmpeg-normalize <source.flac> -ar 16000 -o <source>-norm.wav`，環境中可見
`ffmpeg-normalize 1.15.2`，WAV muxer signature 是 `Lavf57.83.100`。原始素材屬於公開的
LibriSpeech `train-clean-100`；從官方 source FLAC 配合該腳本與相符 toolchain，可高可信度
重建完整音訊。若要求 byte-for-byte 相同，仍需同一 source FLAC、同版 ffmpeg-normalize／
FFmpeg 及相同 normalization defaults，或找到另一份完整副本。

使用者於 2026-09-04 決定不再修復此 WAV，將它視為可重建且可忽略的單一不完整 member。
這項決定只授權後續乾淨 archive 略過該 member，不代表原始 `cph.tgz` 已健康，也不等於立即
刪除原始 archive；原檔仍保留到新 archive 建立與完整性驗證通過。

其中 14 個 Python env 的 1,669 個小型 metadata files 已全部讀完且零 metadata error，
可繼續建立 dossier；但 container 不完整，所以仍不可宣稱整個 `cph.tgz` 可完整還原。

完成版 Git 小型 metadata pass 是 72/72、外接碟寫入 0；三組 clone-family 候選中只有一組
具有相同 HEAD 與 packed-refs，但報告未把任何一組宣告為 exact duplicate clone。

378 筆誤判中有 296 個 Conda `pkgs/` cache，約 3.68 GiB logical／1.995 GiB
archive heuristic；其餘主要是過寬父目錄，以及名稱剛好叫 `env`／`venv` 的原始碼。
不可把誤判父目錄整棵排除。

166 組 SHA-256 完全相同檔案的 archive heuristic 上限約 38.9 GiB，其中約
9.3 GiB 是 venv dependency，已包含在環境清理量。非 venv 部分約 29.5 GiB；
build/cache/binary 子類約 10.5 GiB。環境、cache、build 與 duplicate path 可能重疊，
實際重建前必須建立 path-level 非重疊集合，不能直接相加。

另有一份排除高風險誤判後的 strict-safe cache 清單：1,013 個候選、約 16.7 GiB
logical／9.8 GiB archive heuristic。規則只含 `CACHEDIR.TAG` tree、pip/npm/Conda/Yarn
cache、VS Code server/cache、GPU compute cache、Python bytecode、pytest cache 與明確 OS
測試垃圾；不含 `node_modules`、env、build、log、dataset/model、Trash、AppleDouble 或泛稱
`tmp/swp/core`。後者仍可另行審查，但不可混入 strict-safe 數字。

2026-09-04 在 Mazu 套用帳號活動 gate：NSS 順序是 `files systemd nis sss`，NIS domain
為 `ee.ntu.edu.tw`、server 為 Zeus。仍存在於中央身分來源的 `dvlab`、`chinyi0523`、
`dsnp_student`、`ntuwp`、`ric` 相關候選一律 HOLD，共 138 個 strict-safe 候選、約
4.17 GiB archive heuristic。其餘 875 個 strict-safe 候選屬於中央身分來源已不存續的
owner，約 8.72 GiB logical／5.60 GiB archive heuristic；再加上 3 個 dossier 已完成且
不重疊的環境根目錄，形成目前可納入乾淨 archive 的候選聯集：878 個項目、約 9.75 GiB
logical／6.09 GiB archive heuristic。這仍是重建候選，實際節省以新 archive 完成後為準。
執行時必須保留 strict-safe manifest 的 `path_scope`；`matched_members_only` 不得誤刪父目錄。

## 原生分卷的已驗證能力（2026-09-06）

使用者已指定正式設定為一般等級、512 MiB 字典：`-mx=5
-m0=lzma2:d=512m -mmt=4 -v32g`，不執行字典／壓縮效能比較。
完整性、內容覆蓋與還原驗證仍須執行；以下 128 MiB 是先前功能小測的設定。

官方 7-Zip 26.03 Linux x64 的 `-v` 是原生分卷功能。以合成資料測試 GNU tar
POSIX/sparse/xattrs stream → `7zz a -siCanonical-Home.tar -t7z -mx=5
-m0=lzma2:d=128m -mmt=4 -v1m`，產生三個 `.tar.7z.001/.002/.003`，
整套 `7zz t` 通過；從第一卷 `7zz x -so` 接 GNU tar 還原，逐檔 SHA-256、
中文檔名、空目錄、mode、symlink、hardlink、sparse、測試用 user xattr 一致。
缺少後續卷的負向測試回傳 exit 2。這只證明功能，不證明真實 Home 的壓縮率、
TB 級吞吐量、ACL、跨平台 metadata 或異 UID/GID 還原能力。

此模式是一層 tar 加 7z/LZMA2 壓縮與分卷，不要再先壓縮 tar 再套 7z；
不必先落地完整 tar，也不必先將所有卷合併成巨大檔案才能串流還原。
官方隨附 MANUAL 提醒封裝結束時可能改寫任何卷，故不能將已產生的前卷
提早當成固定、已發布或已驗證結果。Writer 需能 seek 全套輸出卷，不能
直接沿用 zstd → SSH dd 的單一不可回寫輸出管線。所有卷完成後才產生固定
卷清單／SHA-256，分卷本身不等於容錯或中斷續壓。

## Canonical 內容過濾的已驗證注意事項（2026-09-07）

只靠敏感路徑或 `BEGIN PRIVATE KEY` 文字搜尋不足以直接定性。實際資料中，
GitHub token 出現在 Git remote URL、reflog、課程批改 log 與結果檔；保留程式碼與
成績，對備份副本作 token span redaction，比整份丟棄更少損失。JSON 中的 PEM
可能以 escaped newline 儲存；解析器的 header 字串又可能完全沒有 key material。
公開套件的測試私鑰與 README token 範例，已能以 npm/PyPI 官方發佈包及對應
官方原始碼中的 decoded material SHA-256 精確比對。只有比對成功才作公開 fixture
處理；不能只因路徑含 `test` 就判為安全，也不向外部服務傳送來源秘密去驗證。

歷史資料中確實存在名稱以 `.o`／`.swp` 結尾、裡面卻包含完整 Qt 原始碼的目錄。
泛用「副檔名即 build output」規則不能直接套到這類歷史樹；必須保留先前逐項
審核的清理邊界。同內容 dedup 也必須核對對應 current path 確實在最終選中清單；
current 檔案存在但被過濾時，仍須保留歷史內容。

`os.walk(followlinks=False)` 不等於完全不讀 symlink target：實際 Python 原始碼
仍先用 `DirEntry.is_dir()` 分類，才決定不往 symlink directory 遞迴。本次在
canonical overlay 的絕對 Home symlink 上，這一步觸發 autofs 並卡住；核心 stack
是 `autofs_wait`，stat syscall flags 為 0。改用 `os.scandir()` 與顯式
`entry.is_dir(follow_symlinks=False)`，分類和遞迴都不探查連結目標；需讓掃描
錯誤直接失敗，避免 `os.walk` 預設忽略 scandir error。先停止自己受阻的處理程序，
保留 partial evidence，再以修正後的 walker 重建清單；不需為這個程式問題重啟
主機或修改原始 symlink。已留下不允許 target dereference 的 runnable check。

逐檔 archive evidence 應使用 NUL 分隔名稱；SQLite path 用 BLOB 才能保留非 UTF-8
檔名。串流 validator 要讀到 FIFO EOF，避免 tar 結尾 padding 尚在輸出時提早關閉
而使 tee 收到 SIGPIPE。GNU tar 的 `--quoting-style=c` 配合 `LC_ALL=C` 可將消失
路徑的診斷還原成精確 bytes；只接受清單內 current 的 ENOENT，不把讀取中變更
或其他 tar 錯誤混入已授權例外。

## 最小 reconstruction dossier

一般 Python venv 至少保留 Python major/minor、平台、每個 package 的名稱與版本、
原始 requirements／lockfile、`pyvenv.cfg`、`.egg-link` 與 `direct_url.json`。只有 package
name 不足以重建；`pip freeze` 也只是已安裝狀態快照，不是 solver lockfile。
editable、local path 或 VCS dependency 還必須保存 source location、VCS URL／commit，
並確認本地原始碼已被核心程式碼規則保留。

Conda dossier 同時保留：

- `--from-history` 的 structured YAML／JSON，供跨平台重建使用。
- explicit spec，包含完整 package URL／build，供相同平台精確重建使用。
- `conda-meta/history`、平台／channel 與外部 pip package 線索。

這符合 Python 官方將 venv 視為 disposable、不可搬移，以及 Conda 官方區分
cross-platform export 與 same-platform explicit spec 的原則。

## CMake 與 Git

CMake source tree 的 `CMakeLists.txt`、自訂 modules、toolchain、build instructions 與
project-wide `CMakePresets.json` 必須保留。獨立 build tree（可由 `CMakeCache.txt`、
`CMakeFiles/`、產物與 `-S/-B` 關係識別）在上述輸入齊全後可排除；
`CMakeUserPresets.json` 可能含唯一的本機設定，先抽取有用 cache variables 再決定。

Git repository 先保留原貌。後續可由副本執行 `git bundle create --all` 並以
`git bundle verify` 驗證；bundle 保存 reachable refs/objects，但不保存 worktree、index、
stash、repo config、hooks 或 reflog-only/unreachable objects，這些需要另外判斷與保存。

## 執行閘門

1. 帳號活動 gate 優先於技術上的可重建判定。只要帳號仍存在於目前的中央 identity
   database、有 current session/process、近 365 天登入證據，或仍列在現役 roster，
   該帳號的 env、cache、build 與 repo 全部改為 HOLD，除非 owner／管理者另行確認。
2. 備份 snapshot 的 mtime 只能證明備份當時的狀態，不能判斷帳號現在是否活躍。
   `loginctl` 只列 current sessions；`last`/`wtmp` 與 system journal 是 per-host 且受
   retention、權限與記錄設定限制。單一主機沒有紀錄、帳號已從 NSS 消失或 live home
   不可見，都不能單獨證明 inactive；需中央 roster 或跨主機 audit 補證。
3. Git 與 env 小型 metadata pass 已完成；不要因舊 partial 狀態重啟。未經新的明確需求，
   不啟動另一個外接碟掃描。
4. 以完成版 `3 EXCLUDE_AFTER_DOSSIER / 95 HOLD` 為準；舊的 `49/49` 是 metadata 完成前
   套用帳號 gate 的暫定數字，不再作為執行依據。
5. 再納入 Conda cache 與可復現 build tree，產生非重疊 exclusion manifest。
6. 在新 archive 完整建立、完整性驗證、抽樣重建驗證與 SHA-256 manifest 通過前，
   保留原始 archive，不刪除或覆寫。

## 2026-09-05 已執行結果與續作點

長備份 One Touch（UUID `001D-7DC1`）已完成第一批精簡 archive 的重製、完整遞迴驗證、
外接碟寫入、`sync`、唯讀讀回驗證及提交：

- `<long-backup-mount>/backup/dvlab.tgz`：178,168,055,053 bytes，SHA-256
  `301251df790c54eb50b40816fc1d478c12b1f6a63ac2f60b3c56ee93720a0d14`。
- `<long-backup-mount>/backup/yoctol.tgz`：236,762,429,270 bytes，SHA-256
  `35ac9e0424be2ea2cabee021cda915643ae1abcb05e891e2214a99f656af0893`。

兩份成品的 tar/ZIP 巢狀容器、member 數、邏輯位元組與保留內容 SHA 都通過；寫回外接碟後
又從 `.partial` 完整讀回通過，才原子改名並移除被取代的
`backup/dvlab.tgz`（178,497,851,239 bytes）與 `backup/yoctol.tgz`
（245,242,677,163 bytes）。兩份 archive 淨省 8,810,044,079 bytes；提交後 filesystem used
為 581,230,919,680 bytes。`cph.tgz` 保持原始 bytes，不重新包裝；`CADathlon.rar` 內無安全
寫入工具可處理的 3 個候選／48,128 bytes 亦保留。`dsnp_student.tgz`、`ntuwp.tgz`、
`ric.tgz` 因近期帳號 gate 未重製。

正式提交後依使用者要求保留原始外層路徑與檔名：`.cleaned` 與 `backup-clean/` 只可作為 staging
標記，不得成為最後備份結構。兩份已驗證成品已在同一 filesystem 內原子改名回上述
`backup/*.tgz`，大小、mtime 與既有 SHA-256 不變，空的 `backup-clean/` 已移除。

Trash 的完整唯讀核對在 Mazu 恢復後完成：

- `dsnp_student/` 的 6/6 檔案同時與正式 `.tgz`、Trash 舊 `.tar.xz` 內容相同。
- `ntuwp/` 的 62,253 檔中，62,252 檔與正式 `ntuwp.tgz` 內容相同；唯一 size difference
  是 `.cache/Cypress/10.10.0/Cypress/resources/app/node_modules/ajv/dist/ajv.min.js.map`，但 Trash
  版本是 0-byte 空檔，沒有唯一內容。
- `ric/` 的 4,868 檔中，4,867 檔與正式 `ric.tgz` 內容相同；唯一 size difference
  `ric/class/DSnP/1051/fraig_final/b04901036_fraig/src/sat/test/Proof.h` 的 Trash 版本也是
  0-byte 空檔。正式 archive 版本較大，因此不需保留 Trash 空殼。
- `dvlab/` 的 59,027 檔已從頭完成 GNU tar `--compare`，archive 無完整性錯誤。只有 25 個
  `.DS_Store`／`Thumbs.db` 被精簡規則略過，以及
  `dvlab/backup/hschiang/metronic/metronic_v4.6/theme_rtl/admin_3_angularjs/demo/ecommerce_order_invoices.php`
  的 Trash 版本是 0-byte 空檔；其餘 59,001 檔內容相同。
- 3,092,774,912-byte 的舊 `yoctol.tgz` 只讀到外層 `yoctol/` 與宣告
  43,830,855,632-byte 的 `achiang.tgz` header，隨即 unexpected EOF；它只是 `achiang.tgz`
  的短前綴。57,183,305,728-byte 的 `yoctol.2.tgz` 仍是已驗證的完整 `achiang.tgz` 加上
  更短 `cph.tgz` 前綴；兩者都沒有正式精簡 archive 之外的可恢復尾端資料。

上述 7 個 Trash payload 與 7 個 `.trashinfo` 已在 fail-closed 檢查後移除；filesystem used
由 581,230,919,680 降到 465,269,424,128 bytes，實際釋放 115,961,495,552 bytes。之後又移除
Seagate 出廠安裝／教學檔、Spotlight/TemporaryItems metadata、磁碟圖示、AppleDouble 與空 Trash
目錄，另釋放 73,924,608 bytes。整批合計釋放 116,035,420,160 bytes（108.066 GiB），最終 used
為 465,195,499,520 bytes；根目錄只剩 `backup/`。One Touch 已恢復唯讀，沒有
使用者程序占用，近十分鐘 kernel journal 無 USB、exFAT 或 block I/O error。

本批未碰短備份 Ultra Touch。Jonathan 唯一 canonical home
`<short-backup-mount>/backup/<remote-home>/jonathan.tar.zst` 仍是永久保護項，不得列為重複候選。

## 2026-09-07：NFS 備份慢先確認實際連線協商速度

Canonical Home 串流平均約 8.6 MB/s，tar 等待 nfs_file_read；進一步以路由確認使用 eno2，sysfs speed 與 ethtool 均顯示 100 Mb/s 全雙工。介面支援並宣告最高 2500baseT/Full，auto-negotiation 開啟，但當下只協商到 100 Mb/s（理論 12.5 MB/s）。因此不能僅憑 tar 等待 NFS 就歸因於 NAS 磁碟或小檔案；先查實際 link speed。線材、交換器埠或對端設定的具體原因尚未確認；本次只讀診斷，沒有重新協商或中斷備份。

同日追加只讀調查：I226-V（8086:125c rev06），igc 驅動；開機日誌 2026-09-05 15:54:12 為 1000 Mbps，18:33–18:36 多次 link down/up，18:36:25 變為 100 Mbps。EEE 當下 disabled。NetworkManager profile auto-negotiate=no 但 speed=0、duplex 未設；依官方語意為略過 link 設定，不能誤判為強制關閉硬體協商；ethtool live autoneg=on。LLDP 無鄰居；PHY downshift 查詢回 Operation not supported。沒有交換器埠資料，不能確診線材或對端故障，也未執行可能斷線的 cable test／renegotiation。

拓樸診斷補充：ethtool 的 speed 只代表本機與直接相連設備的實體鏈路，不代表整條 NFS 路徑。上游長線降速不會自動使下游網卡也顯示相同速度；應先定位長線所在段。最初即時查詢 NAS 兩個介面的路由都走同一個 100 Mb/s eno2，因此單換 NFS server IP 無法避開當時的本機瓶頸。使用者後續確認所謂長線是 NETGEAR ↔ 防火牆，約 7 公尺；不能用這段長度解釋 Mazu ↔ NETGEAR 的降速，且上游實際協商速度仍未驗證。

後續網路調查：使用者授權重新協商後，`ethtool -r eno2` 成功，2026-09-07 13:03 恢復 1000 Mbps Full Duplex，13:14 複查仍維持，carrier_changes 保持 16。這是恢復連線，尚非根因修復。以 swear02 的 sudo 讀取日誌，確認 9/5 18:36:14 曾報 `exceed max 2 second`；Linux v7.0 `drivers/net/ethernet/intel/igc/igc_main.c` 的 `igc_watchdog_task` 在 1 Gbps 等待 `PHY_1000T_STATUS` 的 `SR_1000T_REMOTE_RX_STATUS`，20 次 100 ms 等待耗盡會報此訊息。它是 Gigabit 接收狀態等待逾時，不能單憑訊息確診線材或 NETGEAR 埠。該時窗先出現 NIC Link Down，NetworkManager 才因 carrier-changed 重連 DHCP；核心沒有 PCIe AER、網卡 watchdog/hang/reset 訊息，所查 sudo 日誌沒有相關網路重設命令，但日誌缺席不代表可排除所有人工操作。I226-V 型號本身也有 Intel 官方 EEE 隨機斷線通報；本機實測 EEE disabled，不應直接套用該根因。

2026-09-07 16:52:23 網路追蹤：重新協商後約 3 小時 49 分鐘仍為 1000 Mbps、full duplex；carrier_changes 仍為 16，13:03 以後的核心日誌無新增斷線或降速。rx_crc_errors 與 rx_errors 均維持原有 1，tx_errors 與 tx_timeout_count 均為 0。這是有時間界線的穩定觀察，不是線材、埠或網卡故障已排除。後續若 NFS 備份仍慢，應重新量測瓶頸，不可繼續沿用已恢復的 100 Mbps 診斷。

2026-09-07 NFS 加速只讀確認：備份掛載 BDI 預讀 128 KiB，rsize/wsize 131072。使用 NFSv4 PUTROOTFH、LOOKUP 到 export、GETATTR maxread/maxwrite，伺服器實際回報兩者均 131072 bytes；因此只改客戶端 rsize 不能突破。RPC 查詢須從保留來源埠發出，否則服務端回 NFS4ERR_PERM，不能誤認成不支援屬性。DSM API 首次登入逾時，重試一次成功並 logout；SYNO.Core.System.Utilization 可取得 NAS 負載（SSH 關閉不代表無監控管道），首次速率欄位可能全零，需多次取樣確認。SYNO.Core.FileServ.NFS read_size/write_size 的原始值未確認單位，不可直接當作 bytes 解讀。只讀檢查未更改伺服器或預讀設定。

2026-09-07 使用者授權後將備份 BDI 的 read_ahead_kb 從 128 暫時調到 1024（當時只有備份掛載共用該 BDI），不中止既有 tar/7zz，也未修改持久設定。調整前 30 秒輸入 50.44 MB/s，後兩個樣本 60.60、58.59 MB/s；READ 在途估計 1.53→5.20/5.09，RTT 3.18→9.59/9.72 ms，重送與逾時 0。表示預讀並行度確實增加；相鄰不同檔案樣本不能當成固定加速比例，延遲同時升高，不宜據此無限加大預讀。BDI 編號不是固定識別，後續必須從實際 mount 重新確認，不能照抄 0:93。原值與測量保存在備份 evidence/readahead-1m-change.json。

使用者決策（2026-09-07）：NFS 預讀不改永久預設，平常維持原本 128 KiB；大型循序讀取／大量傳輸可暫時調到 1024 KiB，工作完成或提前停止後恢復原值。大量小檔案或隨機存取不保證受益，不能只按檔案數自動開啟。切換前確認實際 BDI 及共用掛載；設定影響共用 BDI，並非單檔或單程序專用。目前備份維持 1024 KiB，恢復原值列為收尾必做事項，尚未加入自動切換或自動恢復機制。較高 READ RTT 與較多在途請求同時出現，不代表每個互動操作都慢三倍。

2026-09-08 收尾決策：使用者要求原備份 session 結束，避免兩個 session 同時修改同一 Canonical Home pipeline；這不代表停止壓縮或完成備份。操作交接追加至 `<audit-root>/CANONICAL-HOME-ONE-TOUCH-PLAN.md`。接續者仍須核對完整性／冷讀回／coverage／SHA-256，runner 驗證完成後另有發布步驟；完成或提前停止後恢復 NFS 預讀原值 128 KiB，尚無自動恢復。保留來源及舊 partial、短碟不動。新批次暫存遵循 `<remote-home>/short-backup-handoff/STORAGE.md` 的 SSD 優先規則，不能搬動執行中的 staging／SQLite／FIFO 或把內含 NFS 掛載當本地資料處理。

2026-09-08 約 11:06–11:11 的慢速區段確認為小檔案 NFS 存取：tar 當時讀取
`aigfuzz/*.aag`，同目錄前 1,000 個 regular files 平均 1,227 bytes、中位數
1,163.5 bytes、最大 2,545 bytes。30 秒 tar 輸出 0.919 MB/s，網路接收
0.898 MB/s；短樣本 7zz CPU 約 0.5%，外接碟閒置。三個 5 秒 nfsiostat
差分區間每秒 1,809–3,120 次 RPC、362–423 次 READ，每次 READ 平均
2.825–2.890 KiB，READ RTT 1.338–1.675 ms，queue 約 0.013–0.015 ms，
無新增 READ 重傳或錯誤。nfsiostat 第一份是掛載以來平均，不能當作當下速度。

同一備份 NFS 掛載的一個既有大 tar，以 direct I/O 在 32 GiB offset 循序讀
256 MiB 到 `/dev/null`，耗時 2.556 秒、105 MB/s；這是單一大檔區段測量，
不是全 NAS 磁碟 benchmark。網卡當時 1000 Mbps，10 次 ping 零掉包、平均
0.210 ms，預讀仍為 1024 KiB。這組對照支持當下瓶頸是逐檔 NFS 操作與
小讀取的延遲，而非整條路徑只有約 1 MB/s 頻寬；未取得 NAS 端磁碟／CPU
樣本，不能进一步分拆 server metadata 與隨機磁碟耗時。只做有界讀取，未修改
備份、壓縮參數、NFS 設定或來源內容。

2026-09-08 小檔案分類追加：現役 `current-inputs.nul` 中
`E-Syn/sym_reg/aigfuzz/` 有 1,034,510 個項目：eqn 534,510、aig／sexpr／data／stats
各 100,000、aag／txt 各 50,000。專案 `.gitignore` 明列該目錄；
`sym_reg/data_collect_all.py` 會生成電路、轉換格式、擷取資料並輸出 CSV。
因此此慢速區段是生成的實驗資料與中間產物，不是 Python／Conda dependency。
不能以 `.gitignore` 或有 generator 就推論可全刪：所讀隨機生成呼叫沒有傳入
固定 seed，尚未證明原始樣本可逐 byte 重建。

使用者決策（2026-09-08）：上述 `E-Syn/sym_reg/aigfuzz/` 小檔案原樣保留，
包含原始電路、轉換結果與統計資料；不因可生成、被 Git 忽略或備份緩慢而排除。
維持現役備份清單與來源內容，不再把此批資料列為待刪／待排除候選。
此決策不撤銷既有執行環境 dependency 的排除規則。

2026-09-09 16:14 慢速複查：約 22 小時區間平均 8.8 MB/s 不能當成持續有效
吞吐。當日 kernel 10:06:36 起有 NFS server not responding，所查 10:00 後日誌
首批 server OK 在 15:55:23；兩者相隔約 5 小時 49 分，但未以逐秒傳輸紀錄
證明整段完全零流量。eno2 在 10:51 有四次 down/up、15:55 與 15:56 各一次，
當下已恢復 1000 Mbps，carrier_changes=28。未判定重連是人工操作或自發故障。
備份三份 stderr 不能取代 kernel/NFS 健康檢查；tar 的缺檔仍只有原本九筆。

同次 30 秒實測 tar 輸入 2.081 MB/s、網卡接收 2.036 MB/s；短區間 7zz CPU
約 0.67%。tar 當時等待 NFS readdir，正在讀取 ICC2 2024.09-sp4 安裝樹的
`doc/LM/man/catn/NDM` 手冊，抽樣檔 590 bytes；NFS 差分 READ 平均
0.964–1.110 KiB、每秒約 613–634 次，無新增 READ retrans/errors。
當前小檔案不屬先前原樣保留的 aigfuzz 實驗資料；這不構成移除 EDA 安裝樹
或修改現役 manifest 的授權。此次未停止備份或修改網路。

2026-09-10 加速調查：tar 讀取 `.pth` 區段時，兩次 30 秒取樣都顯示
tar／tee 等待 pipe write、validator 等待 pipe read；7zz CPU 約 262.6%／256.4%，
取樣內沒有新增輸入，取樣間累計輸入仍前進。這是當段下游壓縮限制，不能沿用
前日 NFS 小檔案診斷。主要壓縮 workers 已在高效能核心，各層 CPU quota=max，
7zz／validator VmSwap=0，vmstat 差分 si/so=0，MemAvailable 約 83 GiB。
系統 swap 已用與一次 SSD 忙碌不能直接證明備份換頁；一般使用者 pidstat I/O
回 -1 時，應以具權限的 /proc counter 確認，不能把 -1 當零。

當次預讀仍 1024 KiB、連線 1 Gbps。提高壓縮執行緒／降低壓縮成本可能改善
CPU 區段，但現役 7zz 無可用熱切換，需重跑並做同資料測試；尚未執行參數比較。
NAS 端 tar 主要減少小檔案 NFS 往返，不能解決 Mazu 壓縮限制。提前並行預讀
僅為未測試候選，不得描述為已證實加速；直接複製全部剩餘輸入到 SSD 也不成立，
因 SSD 空間有限且剩餘位元組分母未知。細節在 `<remote-home>/short-backup-handoff/`
的 `PERFORMANCE-20260910.md`。未修改管線、資料保留規則、網路或壓縮設定。
