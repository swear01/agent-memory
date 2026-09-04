---
title: DVLab fleet 的 Synopsys SpyGlass X-2025.06 與 Linux 7 相容性
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-05
updated: 2026-09-05
tags:
  - synopsys
  - spyglass
  - ubuntu
  - linux
  - eda
---

# 已確認根因

- 五台 DVLab 主機升至 Ubuntu 26.04、Linux kernel 7 後，SpyGlass X-2025.06 的 platform selector 與 bundled Perl 只辨識 Linux 2–6，因而回報 `Unknown platform: Linux-7...`。
- 共用設定原本使用 `OPTIMIZE_PERF = snpsmem`；其 `check.Linux4` 在 Ubuntu 26.04 / glibc 2.43 會 SIGSEGV。實測 `runtime`、`memory`、`tcmalloc` 均可運作，部署採用 `runtime`。
- Vendor `sg_shell` 以 `/bin/sh` 執行，卻使用 Bash 專屬的 `${PIPESTATUS[0]}`，在 Ubuntu 的 dash 下會出現 `Bad substitution`。

# 已部署解法

- 共用安裝的 standard environment、主要 bundled Perl 與選配 SpyGlass-VCS bundled Perl 均加入 Linux 7 判斷。
- 共用 `sg_shell` 改由 Bash 執行，allocator 設為 `runtime`。
- 五台主機各提供 `/usr/local/bin/spyglass` 與 `/usr/local/bin/sg_shell` launcher；launcher 會在子程序內載入共用 SpyGlass 與授權環境，再執行 vendor binary。
- 被修改的五個共用原始檔已備份在 `<shared-spyglass-root>/.dvlab-backups/20260905-spyglass-linux7/`。

# 學生使用方式

- GUI 使用小寫命令 `spyglass`；CLI 或批次執行使用 `sg_shell`。
- 正常 login shell 不需先 `source` 設定，不需自行設定 `SPYGLASS_HOME`、授權環境或 compiler；Bash、zsh、fish 都會解析到 `/usr/local/bin` 的 launcher。
- GUI 仍需既有的圖形桌面、VNC 或 X11 forwarding；純文字 SSH 不會自行產生顯示環境。這是遠端 GUI 的一般條件，不是額外的 SpyGlass 設定。
- 若舊 shell 曾快取失效路徑，開新 shell 即可；也可在 Bash/zsh 執行 `hash -r`，或在 tcsh 執行 `rehash`。

# 驗證邊界

- Mazu、Cthulhu、Athena、Valkyrie、Zeus 的 Bash、zsh、fish login shell 都已確認 `spyglass` 與 `sg_shell` 解析到 `/usr/local/bin` launcher。
- 五台均以實際 `sg_shell` Tcl `compile_design` 完成最小 Verilog smoke test 並正常退出；Mazu 另以移除既有 tool 與 license 環境變數的乾淨環境，直接呼叫 launcher，得到 0 fatal、0 errors、0 warnings。
- `spyglass -help` 可正常完成，且不再出現 unknown-platform 或 internal error；驗證後沒有殘留 SpyGlass 程序。
- 未更動授權設定、系統 compiler 或 HAPI process。Synopsys 支援矩陣未正式列出 Ubuntu 26.04，因此大型實際專案仍應保留 smoke-test 觀察。

# 維運規則

- 若同類錯誤重現，先確認 `command -v spyglass`、`command -v sg_shell` 與 kernel major version，再檢查共用 allocator 設定；不要要求學生各自維護 shell profile。
- 完整操作紀錄與現況仍以 DVLab MIS runbook／HackMD 為準；shared memory 只保存可重用的根因與相容性決策。
