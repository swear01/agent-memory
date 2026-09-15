---
title: 遠端 hapi --version 會吃掉 bash -s 剩餘 stdin 並卡住
scope: tools/hapi
project: hapi
tool: hapi
status: active
confidence: high
created: 2026-09-15
updated: 2026-09-15
tags:
  - hapi
  - deploy
  - ssh
  - stdin
---

# 現象

2026-09-14 用 `hapi-ctl.sh update all` 把 `v0.30.4.1` 裝到 mazu 時，本機 `ssh ... bash -s`
超過 14 分鐘沒結束。遠端 `~/.local/bin/hapi` 其實已是 `0.30.4.1`，暫存目錄已空，也沒有還在跑的 `curl`。

根因是安裝腳本用 `printf '%s\n' "$(remote_install_script)" | ssh host bash -s`，最後一行是
`"$INSTALL_DIR/hapi" --version`。compiled binary 會讀 stdin；`bash -s` 的剩餘腳本／空 stdin
被它吃掉後 SSH 就不退出。`assert_hapi_binary_version` 若也在同一支 `bash -s` 裡呼叫
`--version`，會有同樣問題。

# 處理

- 不要重裝已經是目標版本的 NFS 共享 binary；四台 `swear01` 主機共用同一檔，重複寫入會再觸發
  其他機器的 version handoff。
- 遠端核版本改為 `"$INSTALL_DIR/hapi" --version </dev/null`，或改 here-doc／關閉 stdin，
  不要把腳本本體當 `bash -s` 的 stdin 再跑 `--version`。
- 只殺卡住的本機 `hapi-ctl`／`ssh`，不要對 `--started-by runner` children 發 TERM。

截至 2026-09-15，skill 的 `hapi-ctl.sh` / `install-github-release.sh` 仍是這個寫法，尚未修。
`EXPECTED_HAPI_VERSION` 預設仍是 `0.29.0.6`；部署 `0.30.4.1` 必須顯式覆寫。gist／skill 的 expected baseline 也還停在舊版，與 live Unix 7 機不一致。這不擋 `v0.30.4.1` 結案，但下次 `update all` 還會再踩。

# 核 checksum

`grep darwin-arm64 checksums.txt` 會同時命中 `hapi-darwin-arm64.tar.gz` 與
`hapi-desktop-darwin-arm64.zip`。應對單一檔名，或 `sha256sum -c` 前先過濾精確檔名。
此次 tarball 本身的 SHA-256 是通過的。
