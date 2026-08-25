---
title: DVLab fleet 的 Synopsys VCS 與 GCC 12 相容性
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-08-25
updated: 2026-08-25
tags:
  - synopsys
  - vcs
  - gcc
  - ubuntu
  - eda
---

# 已確認根因

- 五台 DVLab Linux 主機均為 Ubuntu 26.04，系統預設 GCC 是 15.2；另已安裝 GCC 12.5、13.4、14.3。
- Synopsys VCS X-2025.06 產生的 C 程式在 GCC 15 編譯時，會因 `implicit declaration of function 'vcs_simpSetEBlkEvtID'` 失敗。GCC 14 起把 implicit function declaration 改為 error，因此這不是 Verilog 原始碼錯誤。
- 這套 VCS 的本機 README 以 GCC 12.3 為預設 compiler；fleet 現有且已通過完整 smoke test 的相容版本是 GCC 12.5。GCC 13.5 並不存在於目前五台套件池，GCC 13 實際版本是 13.4。

# 不可採用的修法

- 只設 `VCS_CC=/usr/bin/gcc-12` 或只傳 `-cc /usr/bin/gcc-12` 不夠：VCS 生成的 Makefile 仍會用 `CC_CG=gcc`，最後可能抓回系統 GCC 15。
- 直接載入 VCS bundled GCC 12.3 的完整環境也不可用：它連帶使用舊 binutils 2.33.1，在 Ubuntu 26.04 / glibc 2.43 會遇到 `.relr.dyn` section type 無法辨識，以及 `libm`／`libmvec` 連結失敗。
- 對 GCC 15 加 `-Wno-error=implicit-function-declaration` 能讓最小案例通過，但只是壓掉 vendor generated code 的錯誤，不是採用的正式修法。
- 不要用 `update-alternatives` 改全系統預設 GCC；其他課程與專案仍應使用 GCC 15.2。

# 已部署解法

- 共用 VCS Bash 設定在 PATH 最前面加入一個共享 `vcs-gcc12` shim directory；其中 `gcc`、`g++` 分別指向各主機的 `/usr/bin/gcc-12`、`/usr/bin/g++-12`，並讓 `VCS_CC` 指向 shim。
- 五台各自提供 `/usr/local/bin/vcs` Bash launcher。任何正常 login PATH 的 Bash、zsh 或 fish 帳號都可直接執行 `vcs`；launcher 在子程序內載入共用 VCS 設定後 exec vendor binary，不會改變使用者 shell 的全域 compiler。
- Valkyrie 原本缺少 license hostname resolution；補齊主機解析後，授權連線與 smoke test 均恢復。記憶不保存授權伺服器 IP、帳密或 license 內容。

# 驗證邊界

- Mazu、Cthulhu、Athena、Valkyrie、Zeus 均以代表性 NIS 學生帳號，不手動 source 設定，直接透過 `/usr/local/bin/vcs` 完成 `-full64` 編譯、連結與模擬。
- 五台 VCS compile path 均實際使用 GCC 12.5；五台 `/usr/bin/gcc` 仍為 15.2。
- Bash、zsh、fish login shell 均解析到 `/usr/local/bin/vcs`。
- 目前只證明最小 Verilog end-to-end smoke test。Synopsys 的 X Foundation 支援矩陣未列 Ubuntu 26.04，較大型 flow 仍可能遇到 shell 或平台相容性問題；license 同時使用量仍受席次限制。

# 維運規則

- 遇到同類錯誤時，先確認 `command -v vcs`、`/usr/bin/gcc -dumpfullversion -dumpversion`，再以 VCS smoke test 確認 generated C 實際 compiler；不要只看使用者 shell 的 `gcc --version`。
- 若 fleet 升級或移除 GCC 12，shared shim 會失效；共用 VCS 設定應 fail closed，不可靜默退回 GCC 15。
- 完整操作紀錄與現況仍以 DVLab MIS runbook／HackMD 為準；shared memory 只保存可重用的根因與相容性決策。
