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
- 登入 `dvlab-env.sh` 會把自己的目錄插在最前，用較晚執行的 `/etc/profile.d/zz-dvlab-apps-path.sh` 提供 `/apps/bin`。`Z20-dvlab-apps.sh` 提供共用登入環境；不可恢復 `/etc/environment` 的 NFS PATH 或從非登入 zshenv 存取 NFS。驗證 login shell 的命令解析及 shared env，功能 smoke 在新 Bash 手動選版。別用 `nobody` 模擬 login（它是 nologin），也別因啟用工具就直接讀出 license／NIS 設定的敏感內容。[Mazu PATH 排序的原始證據](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/mazu/apps-login-path.md)
- Fleet `/apps` 共用 Gaia `192.168.1.200:/volume1/apps` NFSv4，詳細 ACL 和 automount 驗證見 [Mazu 掛載](../mazu/apps-nfs-mount.md)。首次掛載先以一般帳號查可達與讀取，不批次 chmod。沒有額外複製 EDA 樹，也不重建 `/usr/cad`。[四台初次掛載與歷史登入配置收據](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/hapi-fleet/apps-nfs-login.md)
- 選工具版和相容 container 是兩件事。缺 library 先在 launcher 真實環境查依賴，再選相容容器或該版局部 compat lib，避免改系統 SONAME、全域 LD path 或原廠 binary。功能 smoke 通過才發布 `.eda-installed`／`cur`；保留舊版與容器使用的 `/apps/cad`，不可直接改共享 hardlink inode。
- `docker-shell` 用 `--cidfile` 追蹤自建 container、保留 stdin、Bash wait/trap，只清該實例；直接殺 Docker CLI 可能留下服務與 license。ADMS host-network vendor RPC 報 bind error，查短主機名是否走到公網/NAT；已用容器內本機 host mapping 修復。VC Formal 等吃 heredoc stdin 時用 `</dev/null`，否則後續 assertions 可能未執行。Exit 0 需加完成 marker、功能證據與 Error scan。
- vManager 24.03 和 Xcelium 25.03.005 使用本機磁碟專案和 `/apps/eda/cadence/vmanager-session -exec regression.tcl`，state 綁 owner/版本/主機並鎖定，拒絕 symlink/NFS/CIFS。Client profile localhost 不保證 Jetty 無 wildcard listener；走 helper 認證與退出後 stop，驗錯密碼及 ports closed。狀態要自備份，舊 `-local` 已退役。Batch 不等於 GUI 或 coverage merge 通過。
- SAED32 RVT 正式位置 `/apps/cad/cell_library/SAED32_EDK/lib/stdcell_rvt`，不留舊 `SAED_EDK32_28nm` symlink。官方 archives/verification 在新根目錄；minimal synthesis 已驗，P&R/signoff 未驗。使用者不要 SAED 2013 compiler，不再下載／安裝；原 ARM compiler 是另一套。
- ARM compiler 需要 i386 的 `libxt6t64`、`libxtst6`、`libxp6`。Ubuntu 26.04 舊 `libxp6` 依賴過渡 `multiarch-support`，官方 archive 之過渡包不替換 glibc。照原始 pinned 收據核對 SHA，APT simulation 要求不移除／不升級既有套件；實測 bundled JRE 和實際 compiler。Athena 回機後已補該依賴，和 PT 的四台 native smoke 範圍分開。
- TSRI eTAS 要走 `https://etas.tsri.niar.org.tw/tsrisso` 的完整 callback；單登入舊 portal 後 API 401 不代表沒授權。以當下 allow-download 目錄核對版本／sp，學界 SolvNetPlus 當時無軟體下載權。新版 checkout 不能證明舊版。Windows Tanner/PADS、Assura 新裝及 standalone Questa 明確排除，別重跑舊下載佇列。
- ABC/Yosys 為五台各自 host-local 系統安裝，Yosys 指定 `/usr/local/bin/abc`。查一般帳號解析及 synth/ABC pass；GTKWave 與 OSS CAD Suite 並非共用基線。學生入口、版本與 limits 以 `/apps/eda/README.md` 和實測為準。

# PrimeTime 2026.03 原生 libsasl 相容修補（2026-11-12）

- Mazu、Cthulhu、Valkyrie、Zeus 的 `/apps` 均掛載同一 NFS export，只部署一次；四台 Ubuntu 26.04.1／kernel 7.0.0-38／glibc 2.43。Athena 送修，未連線／驗證。
- PT launcher 環境下 `pt_shell_exec` 唯一缺少的外部 library 是 `libsasl2.so.3`。直接 `ldd` 的 libRtxStable／libnffr／libNPI／libnffw／libnsys 已由內附 `linux64/pt/shlib2` 提供；exec2 的 Qt5Core／Qt5Gui 也已內附於 `linux64/pt/shlib/qt/lib`，不是需要另裝的套件。兩個 worker 與六個 FSDB shlib2 DSOs 在內附 runtime 路徑下無缺漏。
- 從 Rocky Linux 8 官方 BaseOS [cyrus-sasl-lib-2.1.27-6.el8_5.x86_64.rpm](https://dl.rockylinux.org/pub/rocky/8.10/BaseOS/x86_64/os/Packages/c/cyrus-sasl-lib-2.1.27-6.el8_5.x86_64.rpm) 抽出 library；RPM SHA-256 `b1a98c3128e57243d2ce7212ced1d8676d4196feac16079e6335413a0b57ca41`。未執行 RPM 安裝、apt 或更動 `/usr/lib`，沒有用 `.so.2` 冒充 `.so.3`。
- 放置於 `/apps/eda/synopsys/primetime/2026.03/lib/compat-rocky8/`：`libsasl2.so.3.0.0`、原 RPM 的 `libsasl2.so.3 -> libsasl2.so.3.0.0` 連結、`COPYING`。Library SHA-256 `7de4d792e5a89acf1d1e38949e0fe9e0a21b42235c21aa0d38fd20d5fb660b30`；所需 GLIBC symbol versions 最高 2.15（2.2.5、2.3、2.3.4、2.4、2.14、2.15），主機提供且 `ldd` 全部解析成功。
- `/apps/eda/env.sh` 的 `synopsys/primetime` → `2026.03` 分支僅加 `libs=lib/compat-rocky8`，由既有 LD_LIBRARY_PATH 邏輯載入。修補後 env SHA-256 `cf2cd1d1d3d3ee7a69b5de4653515c8d083dd2020ef53992df72f5c48f3f3429`。`primetime.sh`、原廠 binary、其他版本、cur、系統 library、license 設定均未修改；PT 重複 source 不重複加入路徑，無效版本拒絕，DC 2026.03 不會載入此相容目錄。
- 管理入口使用 `su_cthulhu`＋sudo；改共享檔前備份至 `<pt-root>/.dvlab-backups/20261112-pt-libsasl/`，保留 `env.sh.before`、未修改的 `primetime.sh.before`、原始 RPM、manifest、`ROLLBACK.md`。相容目錄原本不存在。
- 四台均以一般帳號、新的 `env -i` Bash，公開 `source /apps/eda/synopsys/primetime.sh 2026.03` 入口讀取 CBDK_IC_Contest_v2.5 的 `SynopsysDC/db/typical.db`、IOTDF gate netlist／SDF／絕對 VCD：成功 link 1,171 leaf cells、report_timing、update_power、report_power，exit 0、精確完成 marker、無 Error；Total Power 均 `5.596e-04`。保留 PTE-066（annotated arcs 零 transition）與 PTE-003（停用 timing arcs）兩項 warning，不能稱零 warning。
- Mazu 以 VCS＋Verdi 2026.03、`-debug_access+all`、`tb.v` 不加 `+define+VCD` 產生 `gt.fsdb`，模擬 RESULT PASS（141 cycles／1415 ns）；FSDB SHA-256 `ed12871b6024c0cf1aed7adc93ec04737a8ccea38f28f55d8d3a4d79827dc34d`。原生 PT 從絕對路徑 `read_fsdb`，並完成 update_power／report_power（同 power），exit 0；另有 FSDB 太小而改 regular power analysis 的提示。time_based 的 read_fsdb 只讀 header，須完成 update_power 才證明 event 讀取。不要加舊式 `-P novas.tab pli.a`：2026.03 已拒絕該方式，依 VERDI_HOME 自動整合。
- Mazu 一般帳號使用既有 `docker-shell` 預設 Rocky r7 亦完成相同 VCD 設計／power 測試（exit 0、相同 power）。驗證資料在一般帳號 `~/pt-native-20261112/`：`smoke.tcl`、`dependency-audit.txt`、各 `<host>-native/pt.log`、`mazu-fsdb/pt.log`、`fsdb-sim/auto-pli/`、`docker-check/pt.log`。跨機首次測試漏傳 harness 的 PT_CHECK_* 環境變數，Tcl 報錯但 PT 仍 exit 0；已保留失敗 log，補齊變數重跑並檢查完成 marker／Error 行後才判定通過。
- 還原：先確認沒有新 PT job 使用相容目錄，並核對 env.sh 仍為上述修補後 SHA；以 sudo 將備份 `env.sh.before` 原子複製回 `/apps/eda/env.sh`，再把 `<pt-root>/lib/compat-rocky8` 搬到備份下 `compat-rocky8.disabled`，開新 Bash。若 env.sh 後來另有修改，只移除 PT 2026.03 的 `; libs=lib/compat-rocky8`，不可整檔覆蓋；完整命令在備份的 ROLLBACK.md。還原後 Ubuntu 原生啟動會恢復缺 libsasl，原容器路徑仍可用。未驗證 GUI、分散式分析或大型 signoff 專案。

# 舊版 DC 統一啟動入口（2026-09-28）

- 同學在新的 Bash 登入終端機使用 `source /apps/eda/synopsys/synthesis.sh 2022.12-sp6`，再執行 `dc_shell` 或 `dc_shell -f synth.tcl`。入口自動處理相容環境，不需 sudo、Docker 權限或手動進容器；同一工具換版仍需新終端機。
- 根因：DC 2022.12-sp6 的舊執行檔依賴 `__pthread_unwind@GLIBC_PRIVATE`，五台 Ubuntu 26.04／glibc 2.43 皆無法直接啟動；四台另缺 `/bin/csh`。只補 csh 或沿用既有 libpng12 不足以解決 glibc 問題。
- `/apps/eda/env.sh` 僅對 Ubuntu 上選定的 DC 2022.12-sp6 插入相容命令目錄；`/apps/bin/dc_shell` 與同包命令共用入口，呼叫 `/apps/eda/.compat/dc-2022/run`。後者使用五台既有 `/usr/bin/bwrap` 與從 Rocky r7 映像抽出的共用 rootfs，保留 cwd、HOME、UID／GID、參數與主機檔案路徑。Docker 僅用於管理員一次性準備 runtime，不參與學生啟動。
- 子程序覆蓋相容系統函式庫與設定，保留網路及授權主機解析；passwd／group 資料遮蔽認證欄位。`EDA_DC_RUNTIME=1` 只設在子程序防止遞迴，不得洩漏至父 shell。這是相容執行環境，不是安全隔離承諾。
- 五台 Mazu、Athena、Cthulhu、Valkyrie、Zeus 均以無 Docker 群組的 `nobody` 帳號，經公開 source 入口合成 16-cell counter，寫出 caller-owned mapped Verilog。可重跑 `bash /apps/eda/.compat/dc-2022/check.sh`；涵蓋空格路徑、重複 source、無效版本及同一 shell 換版拒絕。Tcl `analyze` 的檔名含空格時須用 `[list $env(EDA_CHECK_RTL)]`，避免將一個檔名拆成多項。
- Mazu 一般 NIS 帳號另在同一 shell 完成 DC 合成與 VCS 2023.12-sp2 編譯、反相器模擬；VCS 用 `source /apps/eda/synopsys/vcs.sh 2023.12-sp2` 與 `vcs -full64 ...`。DC 互動提示字元、標準輸入及退出亦以無特權帳號驗證。主機 glibc、登入設定與 cur 未改，2026.03 仍解析原入口；不要用舊版失敗紀錄要求同學自行進容器。
- 五台入口／helper／檢查／README SHA-256 已讀回一致。原始檔、更新後 source 及部署紀錄保存在受保護的 `/apps/eda/.admin/dc-entry-20260928/`。Design Vision GUI 與完整學生專案未驗證；其他版本／工具的相容性不能從此結果外推。
- 原有 HackMD 學生指南已就地補上舊版 DC／VCS 指令並修正過時的啟動失敗 QA；API 全文讀回與候選稿一致，閱讀／編輯權限仍為 `signed_in`／`owner`。文件保留學生需要的 source、啟動與 QA，不加入安裝流水帳。

- 五台 Mazu／Athena／Cthulhu／Valkyrie／Zeus 已讀回相同 README、env、docker-shell 與 vManager helper SHA256；本輪測試服務均 MainPID 0，其他使用者容器保留。

## 最後六組版本的功能證據

下列六組皆通過五台公開 source 入口測試；不能外推為所有 GUI、PDK 或 signoff 流程皆驗證。

| 工具與版本 | 已驗證範圍 |
|---|---|
| PrimeSim 2026.03 | Pro／SPICE 分壓 0.5 V，license checkout／checkin |
| VC Formal 2026.03 | counter FPV proven 且 non_vacuous；舊 2023.12 保留 |
| IC Validator 2026.03-sp1-1 | 合格 GDS 0 違規，刻意窄線 GDS 1 違規 |
| IC Workbench 2026.03 | GDS 讀取、編輯、旋轉 90 度、儲存、重開與尺寸；官方範例須先 `cell edit_state 1` |
| ADMS 2026_1_1 | AFS 126 點、Eldo opamp、混合訊號反相器、官方 Solido Wave GUI 範例 |
| vManager VMANAGERAGILE_24.03.004 | Xcelium 25.03.005 兩筆回歸皆 passed，鎖定／拒絕 NFS／停止後關閉 ports |

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/hapi-fleet/eda-install-inventory.md) · [Athena 席位核對](https://github.com/swear01/agent-memory/blob/c3368747fbc1fdde4c15d8fc3c82217500cf2286/machines/hapi-fleet/eda-install-inventory.md)
