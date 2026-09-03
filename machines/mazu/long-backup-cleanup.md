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

378 筆誤判中有 296 個 Conda `pkgs/` cache，約 3.68 GiB logical／1.995 GiB
archive heuristic；其餘主要是過寬父目錄，以及名稱剛好叫 `env`／`venv` 的原始碼。
不可把誤判父目錄整棵排除。

166 組 SHA-256 完全相同檔案的 archive heuristic 上限約 38.9 GiB，其中約
9.3 GiB 是 venv dependency，已包含在環境清理量。非 venv 部分約 29.5 GiB；
build/cache/binary 子類約 10.5 GiB。環境、cache、build 與 duplicate path 可能重疊，
實際重建前必須建立 path-level 非重疊集合，不能直接相加。

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
3. 不與現有 sequential Git metadata pass 並行啟動另一個外接碟掃描。
4. 先完成 98 個 dossier；原始 50 個 READY 還需套用當下的帳號活動 gate，33 個
   editable/local 與 14 個截斷 archive 不自動排除。
5. 再納入 Conda cache 與可復現 build tree，產生非重疊 exclusion manifest。
6. 在新 archive 完整建立、完整性驗證、抽樣重建驗證與 SHA-256 manifest 通過前，
   保留原始 archive，不刪除或覆寫。
