---
title: ICCAD2026B formal stable release contract
project: ICCAD2026B
tags: [release, rust, github-actions, google-drive, rclone, glibc, archive]
status: verified
created: 2026-08-30
updated: 2026-08-30
source_refs:
  - github:swear01/ICCAD2026B#178
  - github:swear01/ICCAD2026B#180
  - github:swear01/ICCAD2026B@stable-2.3
---

# 專案終態

ICCAD 2026 比賽結束後，PR `#180` 只更新封存文件並合進
`main@ad6ca72a`；它沒有改 production code 或 submission artifact。
`stable-2.3@210921a0` 仍是不可變的最終繳交版本，沒有任何後續 branch、未 tag
commit 或 release 取代它。

GitHub repository `swear01/ICCAD2026B` 已設為 private archived/read-only，description
也標明 final submission。封存前已關閉所有剩餘 issue 與 PR；如需再寫入 code、
branch、issue、PR、release 或設定，必須先明確 unarchive repository。

封存文件 merge 後，`main@ad6ca72a` 的完整 baseline、Python 3.13/3.14、Rust、
public BA/determinism、secret scan、GLIBC 2.28 sanity build 與 cold-cache build 均成功；
因沒有新 `stable-*` tag，publish job 正確跳過，沒有產生新的正式 release。

# 正式發布契約

- 正式版本只接受 `stable-*` tag；不得支援 `alpha-*`、`beta-*` 或
  `workflow_dispatch` 的手動版本輸入。錯誤的 `alpha-1.7` release 與 tag 已移除。
- 唯一 submission artifact 是 locked Cargo release build 產生的
  `target/release/regr_fail_bucketing`。Python 只作 BA／parity oracle，不是上傳或
  evaluator 執行入口。
- build 與 cold-cache build 都在 dedicated `mazu-ci-release` self-hosted runner
  上用 rootless Podman 執行 `manylinux_2_28_x86_64`，正式 artifact 的最高 ABI
  必須是 `GLIBC_2.28`。
- `publish` 只對 `stable-*` tag 執行，將同一個已驗證 binary 發到 GitHub Release，
  再用 pinned `rclone:1.75.0` 上傳 Google Drive，並以遠端 listing 核對精確檔名
  `regr_fail_bucketing`。Drive 寫入或 listing 失敗必須讓 release job 失敗。
- runner 不應依賴某位使用者 home 目錄下的 rclone config；OAuth config 與精確
  folder ID 分別由 GitHub Actions secrets `RCLONE_CONFIG`、
  `STABLE_DRIVE_FOLDER_ID` 注入。folder ID 與 Drive link 不得寫入 tracked docs。

# 已驗證基準

`stable-2.3` 指向 merge commit `210921a0`。GitHub Release 與 Drive 的 artifact
大小都是 `2692712` bytes；GitHub SHA-256 為
`77c22ddbba38e8d1d82aca3565b8c116b468e6f5649d53e0fbb72ad1f783e0f9`，獨立下載
核對相同，Drive MD5 也與本機下載一致。release workflow 的 build、cold-cache、
publish／Drive jobs 全部成功。

# 容易重犯的整合錯誤

- 不可從目前 dirty／落後的 local worktree 判斷既有 release 是 Python 或 Rust；應查
  tag 指向、該 tag 的 workflow log 與下載後 ELF markers。歷史上 `alpha-1.6` 是
  67,750,088-byte Nuitka/Python onefile；後來誤發的 `alpha-1.7` 則確實是
  5,413,984-byte Cargo/Rust ELF（含 `rustc` marker、無 Nuitka／Python marker），
  但當時仍把 optional `reqwest`／`tokio` LLM client 編進 binary，所以不符合最終
  「submission 完全沒有 LLM channel」的 contract。`alpha-1.7` 已刪除，不能再當
  正式版本；目前只以 `stable-2.3` 的 no-LLM Rust artifact 為準。
- 「某功能 PR 已 merged」不代表 `main` 已包含它；PR `#170` 曾合到 `develop`，
  導致正式 release 仍未採用 no-LLM 變更。
- 從舊 `develop` 開出的 PR 可能把已淘汰的 release workflow 一起帶回來。PR
  `#179` 曾同時出現大量 conflicts、重新接受 alpha／beta、且缺少 Drive upload。
  不得用整支 merge 或一律選 `theirs` 解決；應以最新 `main` 為底，只移植作者的
  feature commits，並保留 `main` 的 release contract 與 CI tests。
- 判斷這類 PR 時先比較 merge base、列出作者獨有 commits，再用 `git merge-tree`
  或逐顆無提交試套用區分「歷史分支衝突」與「功能本身衝突」。能編譯或單元測試
  通過仍不足以合併；必須再過 current parity、release contract tests、GitHub CI
  與最新 head review。

# Release 前最低核對

1. `scripts/verify-staged`、Rust checks、Rust/Python parity 與 public sanity gates 通過。
2. tag 必須是單調遞增的 `stable-*`，並指向已合進 `main` 的 exact commit。
3. GitHub Release 必須非 draft／非 prerelease，且只有精確命名的 executable。
4. 獨立下載後核對 SHA-256、size、ELF 與最高 GLIBC version。
5. 用 clone-local folder ID 獨立查 Drive，核對精確檔名、size 與 hash；不可只相信
   workflow 的綠燈。
