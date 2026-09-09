---
title: 實驗室五台主機的 qsyn 建置依賴盤點
scope: machines/hapi-fleet
status: active
updated: 2026-09-09
---

2026-09-09 透過五台主機上的 dpkg、實際 executable 及原生編譯確認；此為盤點結果，尚未完成套件統一。

- mazu、cthulhu、athena、valkyrie、zeus 均為 Ubuntu 26.04 amd64。
- 五台 `libgmp-dev`、`libgmp10`、`libgmpxx4ldbl` 均為 `2:6.3.0+dfsg-5ubuntu2`；`libmpfr6` 為 `4.2.2-3`；`pkg-config` 為 `2.5.1-4`。
- 只有 zeus 安裝 `libmpfr-dev 4.2.2-3`。其餘四台缺少該開發套件；有 `libmpfr6` 不表示可以編譯 MPFR 程式。
- 使用 `g++` 編譯包含 `gmpxx.h`、`mpfr.h` 並呼叫 GMP/MPFR 的程式，連結 `-lmpfr -lgmpxx -lgmp`：zeus 編譯、連結、執行通過；其餘四台都在編譯時報 `mpfr.h: No such file or directory`。
- 五台安裝模擬使用相同的上述套件版本：四台各新增一個 `libmpfr-dev`，zeus 無變更，沒有升降級或移除其他套件。這不是實際安裝成功的證據。
- 五台預設 executable：GCC/G++ `15.2.0`、Clang `22.1.8`、CMake `4.4.2`。CMake executable 在 `/usr/local/bin`，不可只讀 dpkg 的 CMake `4.2.3` 就當成實際使用版本。
- mazu、athena、valkyrie、zeus 預設 clang-format 為 `21.1.8`。cthulhu 為 `17.0.6`，透過 alternatives 指向 `/usr/bin/clang-format-17`，沒有安裝 clang-format meta package 或 clang-format-21。
- cthulhu 安裝 `clang-format=1:21.1.6-71`、`clang-format-21=1:21.1.8-6ubuntu1` 的模擬只新增這兩個套件。實際修復仍需處理既有 alternatives 並回查 executable。
- qsyn PR #165 的 CI 使用 clang-format 16。cthulhu、valkyrie、zeus 有 `clang-format-16 1:16.0.6-23ubuntu4`；mazu、athena 沒有，目前 APT 索引也不提供此套件。系統 formatter 一致不等於符合該 PR 的 CI formatter。

管理權限界線：本次登入的 swear01 在五台均無 sudo 權限。既有紀錄指定 mazu 管理使用 swear02，但本次從 valkyrie 以目前 SSH 金鑰登入 swear02 的 mazu、cthulhu、zeus 都遭拒。這不推翻 swear02 本身的管理權限，只表示此工作階段尚無可用管理登入。

另已於一次性 Debian Bookworm 容器驗證 qsyn PR #165 的 `scripts/ensure_gridsynth_deps.sh`（head `25bb383d9911f7b27f9e693f91a808a0ef57da90`）有獨立缺陷：要求三個標頭同目錄，與 Debian/Ubuntu 的 multiarch 配置不符。正常 GMP/MPFR 編譯通過後，腳本仍在 APT 回報套件已安裝後報缺件。補齊實驗室套件不能修正該 repo 腳本；GitHub hosted runner 的容器也不使用實驗室主機套件。
