---
title: ICCAD2026B formal stable release contract
project: ICCAD2026B
scope: projects/ICCAD2026B
tags: [release, rust, github-actions, google-drive, rclone, glibc, archive]
status: verified
created: 2026-08-30
updated: 2026-08-30
source_refs:
  - github:swear01/ICCAD2026B#178
  - github:swear01/ICCAD2026B#180
  - github:swear01/ICCAD2026B@stable-2.3
---

- **Dirty worktree 或新 main 讓人誤判正式交件。** 唯一正式版是 private archived repo 的 `stable-2.3@210921a0`；最後文件 merge 未發布新版。先核 tag、workflow log 與下載 ELF，而非 local file tree。歷史 alpha 有 Python/Nuitka，也曾是帶 LLM 的 Rust ELF；兩者均不可當正式提交。archive 的 code、PR、issue 或 metadata 需要有授權的明確 unarchive。
- **新版只在 develop 或 merge 衝突帶回退役 workflow。** 保留 main 的 `stable-*` 單調 tag、locked Cargo `target/release/regr_fail_bucketing` 與 no-LLM 契約；Python 是 BA/parity oracle。用 merge-base／`git merge-tree` 或無提交試套判斷，從最新 main 只移植作者獨有 feature commits，避免放回 alpha／beta／workflow_dispatch 或漏 Drive publish。合併前執行 `scripts/verify-staged`、Rust/parity/public sanity、CI 和最新 head review。
- **發版 job 綠但 Drive 或 ABI 不符。** Build 與 cold-cache 在 `mazu-ci-release` 用 rootless Podman／manylinux_2_28_x86_64；實際 ELF 最高 GLIBC_2.28。非 draft／prerelease 的 GitHub Release 只上同一個 executable，pinned rclone 1.75.0 再傳 Drive。用 Actions 的 `RCLONE_CONFIG`／`STABLE_DRIVE_FOLDER_ID`，不把憑證、目標 folder ID 寫入 repo。獨立下載查大小、SHA、ELF marker 與 ABI，再驗 Drive 精確檔名、bytes/hash；上傳或 listing fail 讓 job fail。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/ICCAD2026B/stable-release-contract.md)
