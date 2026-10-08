---
title: DVLab fleet 商業與開放源碼 EDA 安裝盤點
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-11
updated: 2026-10-08
tags:
  - eda
  - abc
  - synopsys
  - cadence
  - tsri
  - yosys
---

# SAED32/28nm RVT 補充庫與 compiler 移除（2026-10-08）

- RVT 已依使用者要求併入 `/apps/cad/cell_library/SAED32_EDK/lib/stdcell_rvt`；原有 HVT/LVT、SRAM、tech、references 檔案未改，未變更 cur、預設庫、登入環境或群組成員。舊 `/apps/cad/cell_library/SAED_EDK32_28nm` 已移除，不保留舊路徑連結。
- RVT 官方包 `SAED_EDK32.28nm_CORE_RVT_v_01132015.tar.gz`：1,029,183,436 bytes，MD5 `0112468df6e73a64d5223a34fcbb4269`，SHA-256 `32e8e51396544d4fd8519cfac3854dfc26513998d89eaa2893b0cb06718f7d66`。原始包保留在 `/apps/cad/cell_library/SAED32_EDK/archives/`；安全解壓及 staging→NAS 內容校驗通過。
- Mazu 一般帳號從正式 NAS 路徑使用 `source /apps/eda/synopsys/synthesis.sh 2026.03` 合成 8-bit counter：29 cells、area 104.707330，日誌有 `SAED32_RVT_UNIFIED_SYNTHESIS_OK`。代表 DB 是 `db_nldm/saed32rvt_tt1p05v25c.db`，SHA-256 `9ac1079a5f355e690610b0cb27a0e8241a15c9d920eb94eeac0dce7da2b51dd1`。
- Mazu、Cthulhu、Valkyrie、Zeus 均可讀該 DB 且雜湊相同；Athena 送修中，未測試。完整 P&R／signoff 尚未驗證，不能外推所有 PDK 流程可用。
- SRAM 2015 包 `SAED_EDK32.28nm_SRAM_v_01132015.tar.gz`：2,268,566,367 bytes，MD5 `9c1329ef944957998dd313d64efc04a7`，SHA-256 `e5e5d2f42acb729d3dede8f3c5a64991ea936f7f5138dc881e8a562d06b6a11b`。366 個檔案逐檔 SHA 與既有庫相同，缺漏／差異皆空，不重複展開；原始包保留在 SAED32_EDK 的 archives/。此包不含 compiler 執行程式。
- 獨立 `saed_mc_v2.1.0_30042013.tar.gz` 曾下載及安裝，但原始程式缺 single_32.cfg、Verilog/SPICE 尺寸不一致、Verilog 區塊註解未結束。使用者明確不要此工具；`MC_2.1.0_20130430`、其原始包、本機手冊與測試產物皆已刪除，四台已確認移除。不要重新安裝、下載或排入重試佇列。此決定只針對 SAED 2013 compiler，原有 ARM memory compiler 保留。
- 新增庫目錄 root:student 750、檔案 640。SAED32_EDK 的 README-RVT.md、INSTALLATION-RVT.md、共用 `/apps/eda/README.md` 與既有雲端 EDA 指南已改為統一路徑；原始包、RVT 合成與 SRAM 比對證據移到該根目錄的 archives/、verification/。

- 搬移前後 1,930 個官方檔案逐檔 SHA-256 相同（1,928 個 library 檔案加 SOURCE.sh、CHANGELOG）；既有 13,168 個檔案 inode／大小／mtime 未改。搬移紀錄在 verification/rvt-relocation-20261008.json。之後僅調整新增 SOURCE.sh 的原廠硬編碼路徑為 `/apps/cad/cell_library/SAED32_EDK`，手動 source 已讀回 SAED32_PATH。搬移前 rvt-staging／rvt-nas 證據屬歷史紀錄；重跑使用 verification/rvt-unified/smoke.tcl。

# ARM memory compiler：四台依賴已補，Athena 送修待補（2026-10-08）

- 使用者要求實驗室五台都補相同依賴。Mazu、Cthulhu、Valkyrie、Zeus 已完成系統套件安裝；Athena 未安裝／未驗證，使用者確認送修中，連不上正常。回機後再補相同套件與驗證，不要反覆把送修當成 SSH 故障處理，也不要宣稱五台完成。
- 四台 Ubuntu 26.04 原本均缺 `libxt6t64:i386`、`libxtst6:i386`、`libxp6:i386`。現已安裝 `libxt6t64:i386 1:1.2.1-1.3build1`、`libxtst6:i386 2:1.2.5-1build1`、`libxp6:i386 1:1.0.2-1ubuntu1`；依賴包括 `libice6:i386`、`libsm6:i386`、`libuuid1:i386`。安裝沒有移除或升級既有套件。
- 舊 Ubuntu `libxp6` DEB 有 `Pre-Depends: multiarch-support`，Ubuntu 26.04 的 APT 索引不再提供此過渡套件。從官方 archive 取得 `multiarch-support_2.27-3ubuntu1.6_amd64.deb` 補依賴；它標示 `Multi-Arch: foreign`，內容只有文件與套件資訊，沒有替換 glibc 或新增執行程式。Zeus 原本已有此版本，其餘三台新裝。
- 官方來源：`https://archive.ubuntu.com/ubuntu/pool/main/libx/libxp/libxp6_1.0.2-1ubuntu1_i386.deb`（SHA-256 `972b6d5d8453364b81e78a097b5df4a56f13d9e9d5f010f4282bb0a9a09480ea`）、`https://archive.ubuntu.com/ubuntu/pool/main/g/glibc/multiarch-support_2.27-3ubuntu1.6_amd64.deb`（SHA-256 `70c0efcf6299aeedcf374e54eeb826f9f0e49c3359267aa6e61bf749dde449c1`）。下載核對 SHA 後以 `apt-get -s install` 確認不升級／不移除，再用 `apt-get -y --no-remove install` 安裝上述套件與兩個本地 DEB。
- 查核代表工具：`/apps/cad/cell_library/CBDK_TSMC40_Arm_f2.0/CIC/Memory/sram_sp_hde_rvt_hvt_rvt/r11p2/bin/sram_sp_hde_rvt_hvt_rvt`（compiler r11p2、GUI 7.0.18）。啟動腳本硬編碼 `lib/linux/jre/bin/java`，清空 `JAVA_HOME`／`LD_LIBRARY_PATH`，使用內附的 Intel i386 32-bit Java `1.4.2_04`；不是主機只支援 32-bit，也不是 ARM CPU 執行檔。
- 根因：內附 JRE 的 `lib/i386/libawt.so` 明確連結 `libXt.so.6`、`libXtst.so.6`、`libXp.so.6`。這是舊 GUI runtime 的系統依賴，沒有使用列印仍可能因為缺 `libXp` 載入失敗。用主機的 64-bit 同名 library 不能滿足 32-bit runtime；升級系統 Java 也不會改變硬編碼入口。
- 四台分別通過 `apt-get check`、空的 `dpkg --audit`、套件狀態 `ii`、JRE AWT 的完整 `ldd` 解析，以及上述 compiler `-help`（exit 0）。檢查 AWT 時，要將此 JRE 的 `lib/i386`、`lib/i386/client`、`lib/i386/native_threads` 加到該次 `LD_LIBRARY_PATH`，避免把 JRE 自帶 library 誤報為缺少的系統套件。
- Java 升級僅有初步證據：Cthulhu 主機的 64-bit Temurin Java `21.0.10` 以 `-cp <compiler-root>/lib/gui.zip -Dbasedir=<compiler-root> Main -help` 與 `Main verilog -help` 均 exit 0。GUI 在 headless 測試到建立 JFrame 時出現 `java.awt.HeadlessException`，最後由 timeout 結束；這不是 GUI 成功證據。`jdeps --jdk-internals gui.zip` 也因 `ConstantPool$InvalidEntry` 失敗，不能據此宣稱沒有內部 API 依賴。
- 未修改共享 compiler、原啟動腳本或 Java runtime；完整 GUI、實際 SRAM 產生及 Verilog／LEF 等產物比對未驗證，不能把 `-help` 通過當成可全實驗室切換 Java 21。也沒有同學的實際指令／錯誤訊息，不能宣稱其個案已端到端修復。此處驗證範圍只涵蓋代表工具與指定依賴。

# 舊版 DC 統一啟動入口（2026-09-28）

- 同學在新的 Bash 登入終端機使用 `source /apps/eda/synopsys/synthesis.sh 2022.12-sp6`，再執行 `dc_shell` 或 `dc_shell -f synth.tcl`。入口自動處理相容環境，不需 sudo、Docker 權限或手動進容器；同一工具換版仍需新終端機。
- 根因：DC 2022.12-sp6 的舊執行檔依賴 `__pthread_unwind@GLIBC_PRIVATE`，五台 Ubuntu 26.04／glibc 2.43 皆無法直接啟動；四台另缺 `/bin/csh`。只補 csh 或沿用既有 libpng12 不足以解決 glibc 問題。
- `/apps/eda/env.sh` 僅對 Ubuntu 上選定的 DC 2022.12-sp6 插入相容命令目錄；`/apps/bin/dc_shell` 與同包命令共用入口，呼叫 `/apps/eda/.compat/dc-2022/run`。後者使用五台既有 `/usr/bin/bwrap` 與從 Rocky r7 映像抽出的共用 rootfs，保留 cwd、HOME、UID／GID、參數與主機檔案路徑。Docker 僅用於管理員一次性準備 runtime，不參與學生啟動。
- 子程序覆蓋相容系統函式庫與設定，保留網路及授權主機解析；passwd／group 資料遮蔽認證欄位。`EDA_DC_RUNTIME=1` 只設在子程序防止遞迴，不得洩漏至父 shell。這是相容執行環境，不是安全隔離承諾。
- 五台 Mazu、Athena、Cthulhu、Valkyrie、Zeus 均以無 Docker 群組的 `nobody` 帳號，經公開 source 入口合成 16-cell counter，寫出 caller-owned mapped Verilog。可重跑 `bash /apps/eda/.compat/dc-2022/check.sh`；涵蓋空格路徑、重複 source、無效版本及同一 shell 換版拒絕。Tcl `analyze` 的檔名含空格時須用 `[list $env(EDA_CHECK_RTL)]`，避免將一個檔名拆成多項。
- Mazu 一般 NIS 帳號另在同一 shell 完成 DC 合成與 VCS 2023.12-sp2 編譯、反相器模擬；VCS 用 `source /apps/eda/synopsys/vcs.sh 2023.12-sp2` 與 `vcs -full64 ...`。DC 互動提示字元、標準輸入及退出亦以無特權帳號驗證。主機 glibc、登入設定與 cur 未改，2026.03 仍解析原入口；不要用舊版失敗紀錄要求同學自行進容器。
- 五台入口／helper／檢查／README SHA-256 已讀回一致。原始檔、更新後 source 及部署紀錄保存在受保護的 `/apps/eda/.admin/dc-entry-20260928/`。Design Vision GUI 與完整學生專案未驗證；其他版本／工具的相容性不能從此結果外推。
- 原有 HackMD 學生指南已就地補上舊版 DC／VCS 指令並修正過時的啟動失敗 QA；API 全文讀回與候選稿一致，閱讀／編輯權限仍為 `signed_in`／`owner`。文件保留學生需要的 source、啟動與 QA，不加入安裝流水帳。

# Mazu 舊解壓暫存清理（2026-09-28）

`/var/tmp/eda-staging-20260915/` 的九套舊解壓目錄已清空，整個 staging root 不存在。先清 `fc`、`formality`、`lc`、`primetime`、`spyglass`：與正式版本做完整路徑／類型／大小比對（`rsync -rln --size-only` 零差異）、代表檔 SHA 抽查及引用檢查，Mazu 系統碟觀測減少 192,221,708,288 bytes。`synthesis` 的同一全樹比對五分鐘逾時，並非發現資料錯誤；後續改以套件層級證據清理 `synthesis`、`vcs`、`verdi`、Cadence Xcelium：四套共 18 個原始 tgz 仍在 `/apps/eda/.admin/downloads`，正式版與 `.eda-installed`、先前功能驗證、代表檔 SHA 均有證據，且暫存無程序引用或子掛載。後四套 `df` 另觀測減少 225,131,864,064 bytes；沒有做全樹逐檔內容比對。正式安裝和原始 tgz 清後重查仍在。詳細回執見 Zeus `<admin-home>/playground/storage-audit-20260903/fleet-safe-cleanup-20260928/RESULT.md`。

# TSRI 相同年度 EDA 版本已安裝（2026-09-27）

- 使用者接受替代版本：VCS `2023.12-sp2`、Design Compiler `2022.12-sp6`、Formality `2023.12-sp2`、JasperGold `2024.03p002`。除 VCS 指定 2023.12 系列外，其他只保證與原需求同年份，不能當作精確小版。四個官方 TSRI 下載包先比對 eTAS 提供的檔案大小與 MD5，再解壓至 `/apps/eda/<vendor>/<package>/<version>`；已建立 `.eda-installed`，未改 `cur`。官方壓縮包留在 `/apps/eda/.admin/downloads`。
- Zeus 的 Rocky 8 r7 容器功能驗證：VCS 編譯並執行 SystemVerilog 測試，印出 `VCS_2023_TEST_OK`；DC 以 `/apps/cad/cell_library/CBDK_IC_Contest_v2.5/SynopsysDC/db/typical.db` 成功合成並寫出 mapped Verilog；Formality 以相同 RTL 作 reference/implementation，回報 `Verification SUCCEEDED`；JasperGold `top.p` property proven 100%，且 `jasper_fao` 授權 checkout 成功。這些驗證證明當日所用授權可用，不保證未來授權期限或所有進階功能。
- DC `2022.12-sp6` 在 r7 啟動時缺 `libpng12.so.0`。從 Rocky Linux 8 官方 AppStream `libpng12-1.2.57-7.el8_10.x86_64.rpm` 取出相容 library，放入此版本的 `lib/libpng12.so.0`；`synthesis.sh` 既有邏輯會將該 `lib` 加進 `LD_LIBRARY_PATH`。Zeus Ubuntu 的同名 library 要求 `GLIBC_2.29`，在 Rocky 8 容器不可用。JasperGold 此版在 Rocky 8.9 需 `-allow_unsupported_OS`。
- Mazu、Athena、Cthulhu、Valkyrie、Zeus 均以公開 `source /apps/eda/synopsys/{vcs,synthesis,formality}.sh <version>` 與 `source /apps/eda/cadence/jasper.sh 2024.03p002` 驗證四個 binary 路徑；完整用法見 `/apps/eda/README.md`。五台 source 驗證不是五台各自功能測試；真正編譯、合成與形式驗證在 Zeus 執行。

# Berkeley ABC：五台 Linux 系統安裝（2026-09-26）

- Mazu、Athena、Cthulhu、Valkyrie、Zeus 都在本機 `/usr/local/bin/abc` 安裝 [Berkeley ABC 官方 repo](https://github.com/berkeley-abc/abc) 的 `master` commit `43d923fbb53f7297f218167ac3282d7c251a67bf`；五台執行檔均為 `root:root`、mode `755`、SHA-256 `032d13f9c74cee1c70017fefa2e86ad18a271d1f9a8e66a5e97cb5341726a358`。這是系統命令，非帳號層級安裝。
- 五台均用 `nobody` 執行 `/usr/local/bin/abc -c "version; quit"` 成功；Mazu、Athena、Cthulhu、Valkyrie 的 `swear01` 與 Zeus 的 `swear02` 的 `command -v abc` 均解析到 `/usr/local/bin/abc`。同一執行檔在安裝前以 `i10.aig; strash; print_stats` 驗證，得到 257/224 I/O、2675 AND、50 levels。先前的 `~/.local/bin/abc`、`~/.local/src/abc` 與建置 log 已移除；Zeus 原有 `yosys-abc` 沒有充作此安裝。
- 此版本不會自動更新。更新時從官方 repo 取新 commit 並編譯，再由管理員把 `abc` 安裝到五台各自的 `/usr/local/bin/abc`，記錄新 commit／SHA-256，並重跑跨帳號驗證。學生使用說明見 `/apps/eda/README.md`。

# TSRI eTAS 舊版 EDA 來源查核（2026-09-26）

- 新版下載站的登入必須從 `https://etas.tsri.niar.org.tw/tsrisso` 發起，完成 TSRI SSO 及返回 eTAS 的 callback。直接開 `cs.tsri.niar.org.tw/Security/Login.aspx` 雖可登入，eTAS 的 `/login-state` 與下載 API 仍回 401；正確流程完成後 `/login-state`、`/api/software-download/software-information` 與 `/api/software-download/allow-download` 均回 200。勿因前者的 401 就判定帳號沒有下載權限。
- 當日登入後的 `/api/software-download/allow-download` 清單中，Synopsys `syn` 沒有 `2022.03-sp2`、`vcs` 沒有 `2022.06`、`Formality` 沒有原版 `2023.12`（有 `2023.12-sp2`），Cadence `JASPER` 沒有 `2406`／`2024.06`。這是當日清單快照，重查時以新版站即時目錄為準；不要把不同 service pack 或 VC Formal 當成同版本。
- 新版 eTAS 前端只列 `software-information`、`allow-download`、`package-download-information` 與 `download` 四個軟體下載 API，未見獨立歷史版本入口；舊 `etas.tsri.narl.org.tw/eda/` 當日 HTTP／HTTPS 均連不上。[Synopsys 2026-04-13 台灣學界 SolvNetPlus 使用規則](https://sara.synopsys.com.tw/api/File/Download/PageContent/22/dccbcd59-0c60-4b06-b5ce-1072c35fdcc7.pdf)明定學界帳戶**不能下載軟體工具**，故不能把 SolvNetPlus 當作補齊 TSRI 舊版安裝包的替代入口。若確需舊版，向 TSRI 客服確認是否能提供封存檔。
- Zeus 於 2026-09-26 使用 Rocky r7 容器及已發布的 VCS `2026.03`，成功編譯並執行最小 SystemVerilog 測試，證明當時現行 VCS 授權路徑可用；`2022.06` 的實際 checkout 與 TSRI 現行合約適用性尚未驗證，不能從新版成功外推。取得舊版後須在隔離目錄安裝並做實際編譯測試。

# 安裝結案與維運基線（2026-09-20）

此節取代下方歷史進度；數字與測試為 2026-09-20 收據，不是永久即時狀態。接手先讀結案紀錄，不要重跑舊下載、搬移或解壓佇列。

- 保留的 **37 組目錄項目**已下載、安裝到正式路徑並發布；含既有版本共 **46 個唯一版本**，不重複計算 `cur`。剩餘解壓清單 **24/24** 完成；`continuation-state.json` 的 `next` 為空，`unpublishedCanonical` 為空。
- 明確排除 Windows Tanner／PADS（含文件）、新裝 Assura，以及完整 standalone Questa Sim 的兩卷。保留下載、部分檔與拒絕證據；不要重試。ModelSim、Questa VIP 與 ADMS 內含的 Questa 元件仍保留。Assura 是舊流程的範圍決策，不是整個產品 EOL 的宣告。
- 正式位置仍為 `/apps/eda/<vendor>/<package>/<full-version>`；`source /apps/eda/<vendor>/<tool>.sh @ver` 列版，指定完整版本載入。工具腳本會載入 vendor license，獨立 `license.sh` 保留。同工具換版需乾淨 shell；不要自動載入完整工具環境。
- `.eda-installed`／`cur` 只在正式路徑功能驗證後發布。保留舊版本與 `/apps/cad` 供既有容器；不要修改共用 hardlink inode。
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

## 容器修復與相容性選擇

- `/apps/eda/docker-shell` 用 `--cidfile` 追蹤自己建立的容器，EXIT／TERM／INT 只清理該容器與私有暫存。Docker CLI 背景執行並保留 stdin，再由 Bash builtin `wait` 等待，確保 signal trap 能及時執行。stdin、UID、SIGTERM 143 與容器移除已驗證；只殺 Docker CLI 會留下耗用授權的容器。
- host-network 容器內以 `--add-host "$(hostname):127.0.0.1"` 修正短主機名解析。先前 ADMS 用公開 DNS／NAT 位址作 RPC bind，造成 `cannot assign requested address` 及 Wave 等待；修復後混合訊號與 Wave 五台通過。未改主機 DNS、NFS 或開機設定。
- X11 只複製目前 DISPLAY 的 xauth cookie 到私有 mode 600 檔並唯讀掛入；不使用 `xhost +`。五台真實 Mac XQuartz → SSH → 容器 draw/readback 通過；Custom Compiler 的實際轉送 GUI／OA 儲存重開亦通過，不代表每個工具 GUI 皆通過。
- 預設容器維持 r7。另有 r8-modelsim 32-bit libraries、r9-catapult、r10-virtuoso，以及 `dvlab-eda:rocky8-20260920-r11-icv`。r11 在 r10 加 `snappy`／`libglvnd-opengl`，修復 ICV 的 libsnappy／Workbench 的 libOpenGL 依賴。選工具版本與選相容容器是兩個步驟。
- 測試 harness 若透過 `bash -s` heredoc 輸入，vendor 命令應使用 `</dev/null`，避免 VC Formal 吃掉後續 assertions；process exit 0 不足以证明測試通過。

## vManager 專案服務入口

在相容容器的本機磁碟專案中執行：

```bash
source /apps/eda/cadence/vmanager.sh VMANAGERAGILE_24.03.004
source /apps/eda/cadence/xcelium.sh 25.03.005
/apps/eda/cadence/vmanager-session -exec regression.tcl
```

- helper 使用官方工具建立專案資料庫；狀態放 `$PWD/.vmanager-VMANAGERAGILE_24.03.004`，mode 700、owner 檢查、拒絕 symlink、NFS／CIFS／SMB；file lock 阻止同專案並行。狀態綁版本與主機，不跨主機共用。
- PostgreSQL 僅監聽 localhost；**vManager 的 profile host 設 localhost 不代表服務只綁 loopback**，Jetty 仍 wildcard。helper 使用隨機主密碼與官方 `vmgrconf` 產生的 hash 啟用客戶端認證；錯誤密碼拒絕測試通過。私有密碼、wizard logs 與資料庫不寫入共享記憶。
- helper 自動供應 client 認證，退出停止 server／DB，不建立開機或登入服務。五台鎖定、NFS 拒絕與停止檢查通過；Mazu 兩次生命週期後直接查 DB 得 2 sessions／4 runs，確認真正持久化。初始化失敗的例外不洩露密碼，已有負向測試。
- 本機專案狀態仍需備份；`/var/tmp` 不是永久保存承諾。舊 `-local` 模式自 23.09 移除，不再建議。

## 尚存限制與證據位置

- ADMS 新 Solido SPICE `--spice` 模式缺 LFK；AFS／Eldo／混合訊號／Wave 已可用。沒有繞過授權。
- vManager 24.03 配 Xcelium 25 有 GCC 12.4／libimc fallback 提示，batch 回歸通過，但 GUI／coverage merge 未驗證。PrimeSim 7 個 GDB auto-load 連結指向不可用廠商建置路徑，未盲目補回；核心模擬已通過。
- 其他邊界：Verdi／Verisium 產品 GUI 未驗證；Tessent scan insertion／ATPG、Modus 完整 DFT、Voltus dynamic／IR drop、Sigrity 非 PowerSI II 子工具仍未驗證。source 成功不等於原生 Ubuntu 或所有進階功能可用。
- `/apps` 必須留在早期開機與非登入 shell 之外；五台已確認 `/etc/environment` 與非登入 zshenv 無 `/apps`。本輪未 reboot／remount／改 boot 設定，不因結案重做掛載。
- 結案工作目錄中：`outputs/eda-rollout-status.md`、`outputs/eda-student-guide.md`、`work/eda-rollout/{continuation-state,scope-audit-20260920,deployment-receipt-20260920,live-install-status-20260920}.json`。學生指南同步於 `/apps/eda/README.md`。
- 功能 logs 位於同一 `work/eda-rollout/` 下的 `primesim-vcf-fleet-evidence`、`icv-adms-fleet-evidence`、`adms-wave-fleet-evidence`、`vmanager-fleet-evidence`、`vmanager-session-evidence`、`x11-fleet-evidence`；正式搬移與備份收據於 `/apps/eda/.admin/rollout-20260915/`。不要把原始認證檔案帶入記憶。

# 歷史使用介面與學生文件（2026-09-16）

- 新工具採 `/apps/eda/<vendor>/<package>/<full-version>/`。在 Bash 執行 `source /apps/eda/<vendor>/<tool>.sh <version>`；`@ver` 列已發布版本，`cur` 是可能變動的預設。工具腳本一併載入廠商 license pointer；不要恢復自動選版或把完整工具環境寫入登入檔。
- VCS 與 Verdi 分別使用 `source /apps/eda/synopsys/vcs.sh 2026.03`、`source /apps/eda/synopsys/verdi.sh 2026.03`。需要 FSDB 時在同一 shell 載入兩者；另開 Verdi 行程不會把環境傳給 VCS。這組版本使用 `-debug_access+all`，不要混加舊式 `-P novas.tab pli.a`。
- Verdi 與常用的 nWave 共用 `verdi.sh`；選版後分別用 `verdi &` 或 `nWave &`。已確認兩個命令解析到選定版本的執行檔；本輪未驗證 GUI 顯示。不要將舊版 Xvfb 測試外推到所有新版 GUI。
- 同工具換版需新的乾淨登入終端機；在已載入工具的 shell 內再執行 `bash` 仍會繼承舊環境。專案為重現結果應固定完整版本。
- 使用者明確要求學生文件只保留 source 用法、可直接複製的完整工具指令表、Verdi／nWave 啟動及 EDA 工具相關 QA。不要重複其他文件的 SSH、主機清單、儲存、Docker 教學，也不要塞入文件編寫方法或安裝流水帳。
- 同學的正常使用方式是主機上的工具；容器是相容性與測試環境，不能因先前測試在容器通過，就把容器寫成所有學生的必要步驟。但原生相容性未完成也不能宣稱可用。
- HackMD 原有學生筆記已就地精簡更新，讀回 API `content` 與本機稿完全相同。閱讀權限為 `signed_in`、編輯權限為 `owner`。重新編輯前先匯出比對，更新後再讀回；不另建重複文件，也不把受限筆記識別碼或連結保存到共享記憶。

## 當時驗證與未完成項目（歷史快照；不可當作現況）

- 28 條已發布 source 指令在 Mazu 原生 Bash 的獨立 subshell 載入成功；這只證明入口與環境載入，不代表所有功能、授權 checkout 或 GUI 通過。Design Compiler 2026.03 與 Catapult 2026.1 在本輪後段已出現在 `@ver`，不要沿用較早的未發布清單；版本仍需即時查核。
- Mazu Rocky 8 r7 使用公開 source 入口完成 VCS／Verdi 2026.03 反相器測試，印出 `PASS: inverter` 並產生非空 FSDB；輸出保有原使用者 UID／GID。這不是原生 Ubuntu 測試。
- 同一範例在 Mazu 原生 Ubuntu 啟動 VCS 2026.03 時回報 `/bin/sh: 0: Illegal option -h`。現場讀到 VCS shebang 為 `#!/bin/sh -h`，`/bin/sh` 解析到 dash；本輪僅診斷，未修復。
- Mazu 原生 Calibre 2026.3_27.19 回報 `calibre_vco: 88: Syntax error: redirection unexpected`；ModelSim 2026.1 回報 `vish: error while loading shared libraries: libXft.so.2`。本輪未修復，不能用 source 成功或容器測試通過掩蓋這些問題。
- 證據位於本次文件工作目錄的 `work/concise-source-check.log`、`work/native-vcs-check.log`、`work/vcs-guide-verification.log`、`work/hackmd-concise-receipt.json`，交付稿為 `outputs/eda-student-guide.md`。錯誤是此日期的觀察，後續工作可能修復，接手需重查。

# 歷史盤點（2026-09-14；下列舊路徑與 launcher 行為非現行介面）

- 五台 Ubuntu 26.04（Mazu、Cthulhu、Athena、Valkyrie、Zeus）掛同一份 NAS share `192.168.1.200:/volume1/apps` 於 `/apps`（nfs4）。商業 EDA 樹的公開名字是 `/apps/cad`（從既有學生 tsri 複製，不是 NFS home 上的原樹）。
- Fleet launcher 在 `/apps/bin/{vcs,verdi,spyglass,sg_shell,dc_shell,innovus,jg}`。五台已刪 `/usr/local/bin/{vcs,spyglass,sg_shell}`。當時採登入後直接執行、launcher 自動載入的方式，現已被上面的手動選版取代。當時原因：login 只把 `/apps/bin` 放 PATH 最前並載 license 指向；真正的 `VCS_HOME`／GCC shim／`LD_LIBRARY_PATH` 由 launcher **子行程** source 對應 CIC 再 `exec`，不污染呼叫者的 gcc。不要把完整 `vcs.sh`／`verdi.sh`／`spyglass.sh` 寫進 `~/.bashrc`。
- ICC2、3DIC Compiler、VC Formal **樹在、launcher 不在**（遷移前 fleet 入口本來就只有 vcs／spyglass／sg_shell；本輪 Task 4 清單也沒列這三個）。CIC 已在 `/apps/cad/synopsys/CIC/{icc2,3dicc,vc_formal}.sh`。PrimeRail 有 `primerail.sh` 但安裝樹不存在，不要做 launcher。這三個入口與 FSDB（見下）尚未實作。
- 歷史設定（已撤回，禁止照此恢復）：`/etc/profile.d/Z20-dvlab-apps.sh`（`TMPDIR=/tmp` + source `/apps/etc/env.sh` 只載 license 指向）、`/etc/environment` 把 `/apps/bin` 放 PATH 最前、`/etc/zsh/zshenv` 片段。因為 `dvlab-env.sh` 會把 `/usr/local/bin` 插回 PATH 最前，另有 `/etc/profile.d/zz-dvlab-apps-path.sh`。
- Login 不要 source 完整 `vcs.sh`／`verdi.sh`／`spyglass.sh`。不要印 license 值，不要 cat `license.sh`。不要建 `/usr/cad` symlink。
- Zeus 曾有本機 `/cad`（`/usr/cad` 指向它），只含 Cadence Genus `21.12.000` 與 Conformal `21.20.100`。2026-09-14 已刪 `/cad` 與 `/usr/cad`；沒有遷到 `/apps`，也沒有重建 symlink。其他四台本來就沒有這個目錄。

# 舊共用樹 binary 盤點（2026-09-14）

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

# 開放源碼（host-local；Yosys 已統一）

- 2026-09-26 五台均系統安裝官方 Release `v0.69` 的 Yosys，`yosys -V` 顯示 `0.69+post`（內嵌 Git SHA `143eb14f9cc55d6f8927e68523b0c9d2166ed02c`）。官方 `yosys.tar.gz` SHA-256 為 `6dad6412cae417f5a53e2c943c2aee160162cfc1bdd31669230da1b7e3522571`。五台 `/usr/local/bin/yosys` 均為 `root:root`、mode `755`、SHA-256 `10e6ae08baf36e40f04f2767c6a56521cd5b1030a8847ab68c55ae59d18d427d`；資料目錄為 `/usr/local/share/yosys`。
- 從官方完整原始碼包以 CMake Release 建置，設定 `-DCMAKE_INSTALL_PREFIX=/usr/local -DYOSYS_ABC_EXECUTABLE=/usr/local/bin/abc`，同一份安裝包分發到五台。五台均以 `nobody` 執行 Verilog `synth -top top`，log 確認 ABC pass 與 AND／OR cells；`ldd` 無缺少 library。獨立的系統 `abc` 五台 SHA-256 仍是 `032d13f9c74cee1c70017fefa2e86ad18a271d1f9a8e66a5e97cb5341726a358`。
- 舊版 Cthulhu／Valkyrie `0.35`、Zeus `0.63` 的 `/usr/local/bin/yosys*`、`/usr/local/share/yosys`、`/usr/local/lib/yosys` 已移除後再安裝新版本。舊 `yosys-abc` 已移除；新 Yosys 直接使用 `/usr/local/bin/abc`。原先三台舊版都不受 APT 管理；未找到原始安裝命令或安裝者。
- Mazu、Athena 仍沒有 GTKWave、Icarus、Verilator；Cthulhu 的 APT GTKWave `3.3.126` 未改；Zeus 的 SBY、APT GTKWave `3.3.126`、Icarus `12.0`、Verilator `5.032` 未改。
- 五台系統基線明確排除 GTKWave；不要把它當成共同必裝項。不部署 YosysHQ OSS CAD Suite 進 `/apps`，除非另開任務。

# 舊安裝

- 三份閒置學生家目錄舊 EDA 樹（clare 的 `eda_tools`、HugoChen 的 `eda_tools`、eedave 的 `synthesis_2024.09_linux`）已於 2026-09-14 改名 `.retired-20260914`。觀察期過後才能 `rm -rf`。
- jack0716 的 `tsri` 於 2026-09-14 改名 `tsri.retired-20260914`；使用者於 2026-09-30 明確授權刪除後，退休樹已清除，NFS home 實測釋放 445,900,521,472 bytes。當時已停止的 Valkyrie `opentitan_bug8729` 容器未因此刪除；其舊 bind mount 指向更早已不存在的 `tsri` 路徑。現行共用 EDA 位於 `/apps/cad`、`/apps/eda`，刪後代表檔 SHA-256 仍與刪前一致。完整核對範圍見 `nfs-jack0716-home.md`。
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
- 現行工具腳本會載入各 vendor 的 `license.sh`；license-only 入口仍保留。載入 pointer 不等於 checkout，實際授權成功需執行工具驗證。
- 不要把完整 `vcs.sh`／`verdi.sh` 寫進 login profile：那會改 PATH、`LD_LIBRARY_PATH` 與 VCS GCC shim。若其他 FlexLM 軟體也用 `LM_LICENSE_FILE`，它們可能先去問 TSRI 伺服器而 timeout；Synopsys 偏好 `SNPSLMD_LICENSE_FILE` 以降低這種混用。授權伺服器位址不寫入 memory。

# 維運邊界

- 早期開機與非登入 shell 不可自動觸碰 NFS `/apps`。禁止把 `/apps/bin` 放回 `/etc/environment` 或在 `/etc/zsh/zshenv` source NFS；登入入口與手動工具選版是不同層次。開機修復的最新紀錄見 `ubuntu-gpu-maintenance-plan.md`。
- 下面幾份相容性筆記中的舊 launcher、CIC 路徑與歷史測試只供根因參考；新的操作入口以上方 `/apps/eda` 選版為準。不要依舊筆記把 VCS／Verdi 綁成不可選版的自動環境。
- 保留舊樹供既有使用者與容器，刪除或改動共用 vendor 檔前需確認依賴與 hardlink；本次文件及記憶更新未修改主機工具、授權或系統設定。
