---
title: DVLab fleet 商業與開放源碼 EDA 安裝盤點
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-11
tags:
  - eda
  - synopsys
  - cadence
  - tsri
  - yosys
---

# 現場盤點（2026-09-11）

- 五台 Ubuntu 26.04（Mazu、Cthulhu、Athena、Valkyrie、Zeus）共用同一份 NFS TSRI 安裝樹 `<lab-eda-root>`。該樹在實驗室 NAS 的 NFS home 上，五台都能掛載。
- Fleet 只為 VCS、SpyGlass GUI、SpyGlass CLI 提供 `/usr/local/bin` launcher。Verdi、DC、ICC2、Innovus、JasperGold、VC Formal 必須 source 對應 CIC 腳本。
- Zeus 另有本機 `<zeus-cad-root>`（`/usr/cad` 指向它），只含 Cadence Genus `21.12.000` 與 Conformal `21.20.100`。其他四台沒有這個目錄。

# 共用樹已確認 binary

| 工具 | 版本目錄 | Fleet launcher |
|---|---|---|
| VCS | `vcs/vcs/2025.06` | `/usr/local/bin/vcs` |
| SpyGlass | `spyglass/spyglass/2025.06` | `/usr/local/bin/spyglass`、`sg_shell` |
| Verdi | `verdi/verdi/2025.06` | 無 |
| Design Compiler | `design_compiler/synthesis/2024.09-sp4` | 無 |
| ICC2 | `ic_compiler_2/icc2/2025.06` | 無 |
| 3DIC Compiler | `3dic_compiler/3dicc/2025.06`（binary 名 `3dic_shell`） | 無 |
| VC Formal | `vc_formal/vc_static/2023.12` | 無 |
| Innovus | `innovus/DDI/DDI_25.10.000` | 無 |
| JasperGold | `jaspergold/JASPER/jasper_2025.03` | 無 |

- PrimeRail 的 CIC 腳本指向 `<lab-eda-root>/primerail/primerail`，該目錄不存在。
- Cadence CIC 多數 cshrc 仍假設 `/usr/cad/$VENDOR/$TOOL`。除 Zeus 的 Genus／Conformal 外，這些路徑沒有安裝樹。
- 單元庫在 `<lab-eda-root>/cell_library`：CBDK45、IC Contest、TSMC 018／40／90、SAED32／90。

# 開放源碼（host-local，版本不一致）

- Mazu、Athena：PATH 上沒有 Yosys、GTKWave、Icarus、Verilator。
- Cthulhu：手工 Yosys `0.35`、APT GTKWave `3.3.126`。
- Valkyrie：手工 Yosys `0.35` 與 `yosys-abc`。
- Zeus：手工 Yosys／SBY `0.63`、APT GTKWave `3.3.126`、Icarus `12.0`、Verilator `5.032`。
- 五台系統基線明確排除 GTKWave；不要把它當成共同必裝項。

# 舊安裝仍在 NFS

- 學生家目錄仍有 VCS／Verdi／VC Formal `2023.03-sp2`、Innovus `21.17`、Xcelium `22.03`、ModelSim `2024.1`、DC `2024.09`（非 sp4）。Zeus 本機 `eda.bashrc` 仍指向這些路徑。它們不是 fleet launcher 入口。
- 2026-08-24 審計未刪任何 EDA 檔；Genus 仍需要 Zeus 的 `libpng12-0`。

# NFS 對效能的影響（2026-09-11 Mazu 實測）

- 套件樹是 NFSv4.1，`rsize`／`wsize` 為 32768。當日 Mazu 到 NAS 的鏈路為 1000 Mbps full duplex；這不能外推成永遠 1 Gbps，該介面 2026-09-05 曾降到 100 Mbps。
- VCS `linux64/bin/vcs1`（約 218 MiB）第一次 buffered 讀約 2.10 s（接近當時 1 Gbps 上限），同一檔立刻再讀 0.03 s，代表 page cache 有效。
- 小檔數量才是冷啟動成本：VCS `linux64` 的 `bin`+`lib` 約 6477 個檔、Verdi `linux64` 約 939、SpyGlass home 約 3134。第一次開 GUI 會比之後慢，不是模擬器本身變慢。
- 學生 home 也在同一顆 NAS；預設 `TMPDIR` 未設。真正會拖垮的是把 `csrc`、`simv`、`verdiLog`、FSDB 寫回 NFS home，而不是工具 binary 放 NFS。
- `/tmp` 是本機 tmpfs、`/var/tmp` 是本機 ext4。EDA 暫存應指向這些本機路徑，不要為了效能把整套商業工具複製到五台本機碟。

# 使用與維運

- VCS 要產生 FSDB 時，同一個 shell 必須同時 source `vcs.sh` 與 `verdi.sh`。只靠 `/usr/local/bin/vcs` 會找不到 Verdi PLI。
- 2026 年 TSRI 使用权改 10 月申請。現有 2025.06 目錄名稱不是明年門戶版本的證據。
- 詳細相容性決策見同目錄的 VCS GCC 12、SpyGlass Linux 7、Verdi FSDB 筆記。授權伺服器、IP、license 內容不寫入 memory。
