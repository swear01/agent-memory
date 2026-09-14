---
title: DVLab fleet 商業與開放源碼 EDA 安裝盤點
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-14
tags:
  - eda
  - synopsys
  - cadence
  - tsri
  - yosys
---

# 現場盤點（2026-09-14）

- 五台 Ubuntu 26.04（Mazu、Cthulhu、Athena、Valkyrie、Zeus）掛同一份 NAS share `192.168.1.200:/volume1/apps` 於 `/apps`（nfs4）。商業 EDA 樹的公開名字是 `/apps/cad`（從既有學生 tsri 複製，不是 NFS home 上的原樹）。
- Fleet launcher 在 `/apps/bin/{vcs,verdi,spyglass,sg_shell,dc_shell,innovus,jg}`。五台已刪 `/usr/local/bin/{vcs,spyglass,sg_shell}`。同學登入後打這些名字即可，不必在自己的 shell `source` CIC。原因：login 只把 `/apps/bin` 放 PATH 最前並載 license 指向；真正的 `VCS_HOME`／GCC shim／`LD_LIBRARY_PATH` 由 launcher **子行程** source 對應 CIC 再 `exec`，不污染呼叫者的 gcc。不要把完整 `vcs.sh`／`verdi.sh`／`spyglass.sh` 寫進 `~/.bashrc`。
- ICC2、3DIC Compiler、VC Formal **樹在、launcher 不在**（遷移前 fleet 入口本來就只有 vcs／spyglass／sg_shell；本輪 Task 4 清單也沒列這三個）。CIC 已在 `/apps/cad/synopsys/CIC/{icc2,3dicc,vc_formal}.sh`。PrimeRail 有 `primerail.sh` 但安裝樹不存在，不要做 launcher。這三個入口與 FSDB（見下）尚未實作。
- Login：`/etc/profile.d/Z20-dvlab-apps.sh`（`TMPDIR=/tmp` + source `/apps/etc/env.sh` 只載 license 指向）、`/etc/environment` 把 `/apps/bin` 放 PATH 最前、`/etc/zsh/zshenv` 片段。因為 `dvlab-env.sh` 會把 `/usr/local/bin` 插回 PATH 最前，另有 `/etc/profile.d/zz-dvlab-apps-path.sh`。
- Login 不要 source 完整 `vcs.sh`／`verdi.sh`／`spyglass.sh`。不要印 license 值，不要 cat `license.sh`。不要建 `/usr/cad` symlink。
- Zeus 曾有本機 `/cad`（`/usr/cad` 指向它），只含 Cadence Genus `21.12.000` 與 Conformal `21.20.100`。2026-09-14 已刪 `/cad` 與 `/usr/cad`；沒有遷到 `/apps`，也沒有重建 symlink。其他四台本來就沒有這個目錄。

# 共用樹已確認 binary

路徑相對 `/apps/cad`。

| 工具 | 版本目錄 | Fleet launcher |
|---|---|---|
| VCS | `vcs/vcs/2025.06` | `/apps/bin/vcs` |
| SpyGlass | `spyglass/spyglass/2025.06` | `/apps/bin/spyglass`、`/apps/bin/sg_shell` |
| Verdi | `verdi/verdi/2025.06` | `/apps/bin/verdi` |
| Design Compiler | `design_compiler/synthesis/2024.09-sp4` | `/apps/bin/dc_shell` |
| ICC2 | `ic_compiler_2/icc2/2025.06` | 無（CIC：`synopsys/CIC/icc2.sh`） |
| 3DIC Compiler | `3dic_compiler/3dicc/2025.06`（binary 名 `3dic_shell`） | 無（CIC：`synopsys/CIC/3dicc.sh`） |
| VC Formal | `vc_formal/vc_static/2023.12` | 無（CIC：`synopsys/CIC/vc_formal.sh`） |
| Innovus | `innovus/DDI/DDI_25.10.000` | `/apps/bin/innovus` |
| JasperGold | `jaspergold/JASPER/jasper_2025.03` | `/apps/bin/jg` |

- PrimeRail 的 CIC 腳本指向 `/apps/cad/primerail/primerail`，該目錄不存在。
- Cadence CIC 多數 cshrc 仍假設 `/usr/cad/$VENDOR/$TOOL`。Zeus 本機 Genus／Conformal 已刪，這些路徑現在五台都沒有安裝樹。
- 單元庫在 `/apps/cad/cell_library`：CBDK45、IC Contest、TSMC 018／40／90、SAED32／90。

# 開放源碼（host-local，版本不一致）

- Mazu、Athena：PATH 上沒有 Yosys、GTKWave、Icarus、Verilator。
- Cthulhu：手工 Yosys `0.35`、APT GTKWave `3.3.126`。
- Valkyrie：手工 Yosys `0.35` 與 `yosys-abc`。
- Zeus：手工 Yosys／SBY `0.63`、APT GTKWave `3.3.126`、Icarus `12.0`、Verilator `5.032`。
- 五台系統基線明確排除 GTKWave；不要把它當成共同必裝項。不部署 YosysHQ OSS CAD Suite 進 `/apps`，除非另開任務。

# 舊安裝

- 三份閒置學生家目錄舊 EDA 樹（clare 的 `eda_tools`、HugoChen 的 `eda_tools`、eedave 的 `synthesis_2024.09_linux`）已於 2026-09-14 改名 `.retired-20260914`。觀察期過後才能 `rm -rf`。
- jack0716 的 `tsri` 已於 2026-09-14 改名 `tsri.retired-20260914`（Valkyrie `opentitan_bug8729` 已 `docker stop`，未 `rm`）。觀察期同日，不要 `rm -rf`。
- 那些舊樹不是 fleet launcher 入口。Zeus 本機 `eda.bashrc` 已隨 `/cad` 刪除。
- Zeus 的 `libpng12-0` 仍安裝（舊 Genus 依賴）；工具樹已刪，套件可留到另案。

# NFS 對效能的影響（2026-09-11 Mazu 實測；login 現況 2026-09-14）

- 套件樹是 NFSv4.1，`rsize`／`wsize` 為 32768。當日 Mazu 到 NAS 的鏈路為 1000 Mbps full duplex；這不能外推成永遠 1 Gbps，該介面 2026-09-05 曾降到 100 Mbps。
- VCS `linux64/bin/vcs1`（約 218 MiB）第一次 buffered 讀約 2.10 s（接近當時 1 Gbps 上限），同一檔立刻再讀 0.03 s，代表 page cache 有效。
- 小檔數量才是冷啟動成本：VCS `linux64` 的 `bin`+`lib` 約 6477 個檔、Verdi `linux64` 約 939、SpyGlass home 約 3134。第一次開 GUI 會比之後慢，不是模擬器本身變慢。
- 學生 home 也在同一顆 NAS。Login 已設 `TMPDIR=/tmp`。真正會拖垮的是把 `csrc`、`simv`、`verdiLog`、FSDB 寫回 NFS home，而不是工具 binary 放 NFS。
- `/tmp` 是本機 tmpfs、`/var/tmp` 是本機 ext4。EDA 暫存應指向這些本機路徑，不要為了效能把整套商業工具複製到五台本機碟。

# License 環境變數

- 共用 `license.sh` 只 `export` FlexLM 指向變數，不呼叫 `lmstat`、不連授權伺服器、不 checkout。Synopsys SCL 文件把這種變數稱為 pointer，並允許寫進 `.bashrc`；實際 feature checkout 發生在工具啟動時。
- Login 只 source `/apps/etc/env.sh`（license 指向）。各 `/apps/bin` launcher 會在子行程 source 對應 CIC 腳本，因而也會載入 license。預設只載入 `license.sh` 不會多佔席次。
- 不要把完整 `vcs.sh`／`verdi.sh` 寫進 login profile：那會改 PATH、`LD_LIBRARY_PATH` 與 VCS GCC shim。若其他 FlexLM 軟體也用 `LM_LICENSE_FILE`，它們可能先去問 TSRI 伺服器而 timeout；Synopsys 偏好 `SNPSLMD_LICENSE_FILE` 以降低這種混用。授權伺服器位址不寫入 memory。

# 使用與維運

- 學生登入即可打 `vcs`／`verdi`／`spyglass`（以及 `sg_shell`、`dc_shell`、`innovus`、`jg`）。`command -v` 應為 `/apps/bin/...`。學生講義：[DVLab 商業 EDA 使用說明](https://hackmd.io/ap5l9P9-Rp2bWEujO9htcw)（HackMD `ap5l9P9-Rp2bWEujO9htcw`，`signed_in` 可讀、owner 可寫）。
- FSDB 仍須**同一個行程**同時帶 VCS 與 Verdi CIC。`/apps/bin/verdi` 已 source `verdi.sh`；`/apps/bin/vcs` 只 source `vcs.sh`。先執行 `verdi` 再執行 `vcs` 無效：兩個子行程，Verdi PLI 帶不過去。根因見 `verdi-x2025-fsdb-integration.md`：`vcs.sh` 的 `VERDI_PLI` 指向不存在的 `design_compiler/verdi/...`，要靠 `verdi.sh` 設 `VERDI_HOME`。遷移計畫寫明：login 禁止 source 完整 CIC；若 `$fsdbDumpfile` 失敗，只改 `/apps/bin/vcs` 內部同時 source `vcs.sh` 與 `verdi.sh`。Task 6 smoke 沒測 FSDB，所以還沒改。補 launcher 時維持 `set +u` 包住 CIC source（`machines/mazu/apps-cad-cic-nounset.md`）。
- 2026 年 TSRI 使用权改 10 月申請。現有 2025.06 目錄名稱不是明年門戶版本的證據。不等 10 月 TSRI 新包；本計畫只搬現有 2025.06 樹。
- 詳細相容性決策見同目錄的 VCS GCC 12、SpyGlass Linux 7、Verdi FSDB 筆記。授權伺服器、IP、license 內容不寫入 memory。
