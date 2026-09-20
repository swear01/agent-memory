---
title: DVLab fleet 商業與開放源碼 EDA 安裝盤點
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-11
updated: 2026-09-20
tags:
  - eda
  - synopsys
  - cadence
  - tsri
  - yosys
---

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
- 現行工具腳本會載入各 vendor 的 `license.sh`；license-only 入口仍保留。載入 pointer 不等於 checkout，實際授權成功需執行工具驗證。
- 不要把完整 `vcs.sh`／`verdi.sh` 寫進 login profile：那會改 PATH、`LD_LIBRARY_PATH` 與 VCS GCC shim。若其他 FlexLM 軟體也用 `LM_LICENSE_FILE`，它們可能先去問 TSRI 伺服器而 timeout；Synopsys 偏好 `SNPSLMD_LICENSE_FILE` 以降低這種混用。授權伺服器位址不寫入 memory。

# 維運邊界

- 早期開機與非登入 shell 不可自動觸碰 NFS `/apps`。禁止把 `/apps/bin` 放回 `/etc/environment` 或在 `/etc/zsh/zshenv` source NFS；登入入口與手動工具選版是不同層次。開機修復的最新紀錄見 `ubuntu-gpu-maintenance-plan.md`。
- 下面幾份相容性筆記中的舊 launcher、CIC 路徑與歷史測試只供根因參考；新的操作入口以上方 `/apps/eda` 選版為準。不要依舊筆記把 VCS／Verdi 綁成不可選版的自動環境。
- 保留舊樹供既有使用者與容器，刪除或改動共用 vendor 檔前需確認依賴與 hardlink；本次文件及記憶更新未修改主機工具、授權或系統設定。
