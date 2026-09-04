---
title: Mazu 長備份可重建資料清理邊界
scope: machines/mazu
machine: mazu
tags: [backup, archive, venv, conda, git, cmake, deduplication]
status: active
created: 2026-09-04
updated: 2026-09-04
---

# Mazu 長備份可重建資料清理邊界

## 使用者原則

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
已宣告的 readable file bytes 約 78.291 GiB logical；因此不是整包都無法讀，而是上述 WAV
為部分資料、其後資料缺失或不可知。工具訊息中的 467,968 bytes 只是當下 tar entry 尚需的
uncompressed data，不能推論整包只缺這麼多。現有證據無法區分是原始建立中斷、複製中斷，
或來源檔本來就壞。

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
