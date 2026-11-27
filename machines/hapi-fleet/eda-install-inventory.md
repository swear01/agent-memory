---
title: DVLab fleet 商業與開放源碼 EDA 安裝盤點
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-11
updated: 2026-11-28
tags:
  - eda
  - abc
  - synopsys
  - cadence
  - tsri
  - yosys
---

# Fleet EDA 選版、共享 NFS、授權與驗證

- **同一個 Synopsys pointer 回 socket error。** Athena 的 Calibre 隨附 Siemens lmstat 曾失敗，`/apps/eda/cadence/IC/cur/tools.lnx86/bin/lmstat -c "$SNPSLMD_LICENSE_FILE" -f Design-Compiler` 與 `-f DC-Ultra-Opt` 查詢均成功。先換這支 FlexNet lmstat 複查席位，再用實際工具證明 checkout。
- 正式目錄 `/apps/eda/<vendor>/<package>/<full-version>`。乾淨 shell 用 `source /apps/eda/<vendor>/<tool>.sh @ver` 列版，再選 full version；舊 CIC 腳本是固定版。登入只載必要入口或 license pointer。License checkout 要實際跑功能，不能只看 source／lmstat。版本切換避免疊加 PATH、LD_LIBRARY_PATH 與 VCS GCC shim。
- `/apps` 由多台共用 NFS，更新只做一次，逐台以一般帳號和正式路徑驗證。`/etc/environment`、早期 service、非登入 zshenv 不放 NFS PATH。模擬產物、FSDB、simv、csrc 和 Verdi log 用本機 `/tmp` 或 `/var/tmp`；工具 cold start 的小檔延遲不是 simulation 計算本身變慢。不因舊快照批次 remount 或 reboot。
- 登入 `dvlab-env.sh` 會把自己的目錄插在最前，用較晚執行的 `/etc/profile.d/zz-dvlab-apps-path.sh` 提供 `/apps/bin`。`Z20-dvlab-apps.sh` 提供共用登入環境；不可恢復 `/etc/environment` 的 NFS PATH 或從非登入 zshenv 存取 NFS。驗證 login shell 的命令解析及 shared env，功能 smoke 在新 Bash 手動選版。別用 `nobody` 模擬 login（它是 nologin），也別因啟用工具就直接讀出 license／NIS 設定的敏感內容。[Mazu PATH 排序的原始證據](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/mazu/apps-login-path.md)
- Fleet `/apps` 共用 Gaia `192.168.1.200:/volume1/apps` NFSv4，詳細 ACL 和 automount 驗證見 [Mazu 掛載](../mazu/apps-nfs-mount.md)。首次掛載先以一般帳號查可達與讀取，不批次 chmod。沒有額外複製 EDA 樹，也不重建 `/usr/cad`。[四台初次掛載與歷史登入配置收據](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/hapi-fleet/apps-nfs-login.md)
- 選工具版和相容 container 是兩件事。缺 library 先在 launcher 真實環境查依賴，再選相容容器或該版局部 compat lib，避免改系統 SONAME、全域 LD path 或原廠 binary。功能 smoke 通過才發布 `.eda-installed`／`cur`；保留舊版與容器使用的 `/apps/cad`，不可直接改共享 hardlink inode。
- `docker-shell` 用 `--cidfile` 追蹤自建 container、保留 stdin、Bash wait/trap，只清該實例；直接殺 Docker CLI 可能留下服務與 license。ADMS host-network vendor RPC 報 bind error，查短主機名是否走到公網/NAT；已用容器內本機 host mapping 修復。VC Formal 等吃 heredoc stdin 時用 `</dev/null`，否則後續 assertions 可能未執行。Exit 0 需加完成 marker、功能證據與 Error scan。
- vManager 24.03 和 Xcelium 25.03.005 使用本機磁碟專案和 `/apps/eda/cadence/vmanager-session -exec regression.tcl`，state 綁 owner/版本/主機並鎖定，拒絕 symlink/NFS/CIFS。Client profile localhost 不保證 Jetty 無 wildcard listener；走 helper 認證與退出後 stop，驗錯密碼及 ports closed。狀態要自備份，舊 `-local` 已退役。Batch 不等於 GUI 或 coverage merge 通過。
- SAED32 RVT 正式位置 `/apps/cad/cell_library/SAED32_EDK/lib/stdcell_rvt`，不留舊 `SAED_EDK32_28nm` symlink。官方 archives/verification 在新根目錄；minimal synthesis 已驗，P&R/signoff 未驗。使用者不要 SAED 2013 compiler，不再下載／安裝；原 ARM compiler 是另一套。
- ARM compiler 需要 i386 的 `libxt6t64`、`libxtst6`、`libxp6`。Ubuntu 26.04 舊 `libxp6` 依賴過渡 `multiarch-support`，官方 archive 之過渡包不替換 glibc。照原始 pinned 收據核對 SHA，APT simulation 要求不移除／不升級既有套件；實測 bundled JRE 和實際 compiler。Athena 回機後已補該依賴，和 PT 的四台 native smoke 範圍分開。
- TSRI eTAS 要走 `https://etas.tsri.niar.org.tw/tsrisso` 的完整 callback；單登入舊 portal 後 API 401 不代表沒授權。以當下 allow-download 目錄核對版本／sp，學界 SolvNetPlus 當時無軟體下載權。新版 checkout 不能證明舊版。Windows Tanner/PADS、Assura 新裝及 standalone Questa 明確排除，別重跑舊下載佇列。
- ABC/Yosys 為五台各自 host-local 系統安裝，Yosys 指定 `/usr/local/bin/abc`。查一般帳號解析及 synth/ABC pass；GTKWave 與 OSS CAD Suite 並非共用基線。學生入口、版本與 limits 以 `/apps/eda/README.md` 和實測為準。

# PrimeTime 2026.03：native `libsasl2.so.3` 缺漏

- 用真正的 launcher 環境查 `pt_shell_exec`；工具自帶 `shlib2` 與 Qt5，裸 `ldd` 顯示缺少內附項目時不能直接當作系統缺件。Ubuntu/glibc 2.43 上的外部缺件是 `libsasl2.so.3`。從已核對 SHA 的 Rocky Linux 8 官方 `cyrus-sasl-lib-2.1.27-6.el8_5.x86_64.rpm` 抽 library、SONAME symlink 和 COPYING，放 `/apps/eda/synopsys/primetime/2026.03/lib/compat-rocky8`，只讓 `/apps/eda/env.sh` 的 `synopsys/primetime` 2026.03 分支設定 `libs=lib/compat-rocky8`。不要裝 Rocky RPM 到宿主、改 vendor binary 或用 `.so.2` 代 `.so.3`。核對重複 source、不合法版本、DC 不載該目錄。
- 共享 `/apps` 只變更一次。以一般帳號、新的 `env -i` Bash 走正式 `source /apps/eda/synopsys/primetime.sh 2026.03`，核對相符 DB、mapped netlist/SDF/VCD、真正的 link 與 timing/power 完成 marker，再查 Error。PT 的 Tcl error 可能仍 exit 0；漏傳 smoke harness `PT_CHECK_*` 不能算通過。四台原生 VCD、Mazu 原生 FSDB／Rocky 容器的驗證分開；當時未連 Athena、未驗 GUI／分散式／大型 signoff。
- VCS＋Verdi 2026.03 走 `-debug_access+all`、`VERDI_HOME` 自動 PLI；舊 `-P novas.tab pli.a` 已拒絕。FSDB 用絕對路徑 `read_fsdb`，time_based 讀 header 不等同讀完整 events。再跑 `update_power` 和 `report_power`，核對是否落回 regular power analysis，保留 PTE-066/PTE-003 警告。
- 用 `<pt-root>/.dvlab-backups/20261112-pt-libsasl/` 中 `ROLLBACK.md`、manifest 和原檔。先查沒有 job 使用 compat lib 且核對 live env SHA；只有完全符合當時修補版才整檔原子回復。若另有修改，只撤該分支的 `libs=lib/compat-rocky8`，再移走 compat 目錄並用新 Bash 驗；恢復原生缺 libsasl 時用已驗的 Rocky 容器。

# 舊版 DC：glibc 不相容與正式入口

- 新 Bash `source /apps/eda/synopsys/synthesis.sh 2022.12-sp6` 後執行 `dc_shell -f synth.tcl`。舊 binary 在 Ubuntu 26.04／glibc 2.43 的 `__pthread_unwind@GLIBC_PRIVATE` 缺件無法靠補 csh/libpng12 修。正式入口只在該版用 `/apps/eda/.compat/dc-2022/run`；既有 `bwrap` 與 Rocky r7 rootfs 保留 caller cwd/HOME/UID/GID 和網路授權。學生不需要 sudo、Docker 或自行開容器，這是相容環境而非安全隔離。
- `EDA_DC_RUNTIME=1` 只在子程序防遞迴；passwd/group 遮蔽認證欄位。用 `/apps/eda/.compat/dc-2022/check.sh` 驗空格路徑、重複 source、無效版和同 shell 換版被拒，確認 mapped Verilog 由 caller 擁有。Tcl `analyze` 對空格檔名用 `[list $env(EDA_CHECK_RTL)]`。一般帳號還需實際編譯和模擬，不能以 command resolution 代替。原始改動備份查 `/apps/eda/.admin/dc-entry-20260928/`；Design Vision GUI 和其他版本需各驗。

- 部分工具的特定版本有公開 source 的 smoke，不能推論全部 GUI/PDK/signoff。如果 IC Workbench 官方範例只能選取、不能編輯，先查 `cell edit_state 1`。VC Formal smoke 要有 proven 且 non_vacuous；IC Validator 至少有正常及刻意違規 GDS 對照。

[PrimeTime、DC 版本精確雜湊、安裝範圍和六組功能測試收據](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/hapi-fleet/eda-install-inventory.md) · [整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/hapi-fleet/eda-install-inventory.md) · [Athena 席位核對](https://github.com/swear01/agent-memory/blob/86f38610525e57a230a7c3b87405fea76530bb1c/machines/hapi-fleet/eda-install-inventory.md)
