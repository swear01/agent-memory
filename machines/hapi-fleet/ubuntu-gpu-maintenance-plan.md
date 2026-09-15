---
title: 實驗室 Ubuntu、NVIDIA 與 CUDA 分階段維護計畫
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-08-22
updated: 2026-09-15
tags:
  - ubuntu
  - nvidia
  - cuda
  - bios
  - maintenance
---

# 已確認決策

- Zeus 使用老舊 Supermicro `X10DAI`，BIOS `2.0`（2016）；使用者明確決定不更新 BIOS。
- 五台實驗室主機已完成 Ubuntu `26.04 LTS` 升級；2026-09-15 驗證 kernel 為 `7.0.0-31-generic`、NVIDIA driver 為 `580.178.04`，GPU 正常。
- Athena 在 BIOS `1836` 後仍有反覆 kernel panic／自動重開機；原因尚未完全確認，且 BIOS、microcode 與 Intel Default Settings 均已處理，不可再把它們列為尚未嘗試的修復。
- Zeus 的 GTX 1080 Ti 是 Pascal `sm_61`，必須保留 CUDA `12.9`；CUDA 13.x 不可套用到 Zeus。
- Mazu、Cthulhu、Athena、Valkyrie 統一使用 CUDA Toolkit `13.3 Update 1`；Zeus 單獨維持 CUDA `12.9` 相容線。
- Zeus 的 CUDA `12.9` Toolkit 尚未安裝；目前只有 R580 driver。不可把 `nvidia-smi` 顯示的 CUDA capability 當成已安裝 Toolkit。

# 最新維護狀態（2026-09-15）

- 五台已移除早期系統 PATH 中的 NFS 路徑；四台 Snap AppArmor 均恢復，Zeus 沒有 Snap。APT 自動更新停用，四台 Snap refresh hold。
- Mazu `/home` 已切換為 fstab + systemd automount，autofs.service 已停用；五台 root 密碼登入均鎖定。新設定通過在線檢查，沒有重新開機驗證。
- 使用者後續明確禁止重開；Mazu 使用者工作是在另獲明確授權後結束，不能把允許暫時斷線解讀為可丟棄未保存的計算進度。
- 後續帶日期的段落保留調查過程；其中「尚未恢復」或「尚未切換」描述當時狀態，最新結果以本節與末尾完成紀錄為準。

# CUDA 版本判斷（2026-08-22）

- NVIDIA 官方目前最新 CUDA Toolkit 是 `13.3 Update 1`；CUDA 13.x 支援 Turing 及更新架構，Pascal 的最後 Toolkit 支援線是 CUDA 12.x。
- Ada 主機已依後續使用者決策安裝 CUDA `13.3 Update 1`；四台各有 37 個 CUDA 套件，`cuda-toolkit-13-3=13.3.1-1`，沒有其他 CUDA Toolkit 分支殘留。
- Zeus 已升到 Ubuntu `26.04`，使用者接受 CUDA `12.9` 在該 OS 上未列官方驗證矩陣的風險；安裝計畫仍需使用獨立的 Toolkit 來源，不加入 CUDA 13.x repository。
- Athena 目前不是 live freeze；freeze recurrence 仍是立即停止條件。

# Athena freeze 現況複查（2026-08-22 17:17）

- Athena 目前不是 live freeze 狀態：SSH 可用、目前 boot 自 `16:13:19` 已運行約 `1:03`，`nvidia-smi` 正常，RTX 4090 約 `37°C`、閒置。
- 本次 current boot 與上一個異常 boot 都沒有新的 `NVRM Xid`、GPU 掉線、PCIe AER、MCE、watchdog、OOM、NVMe I/O 或 ext4 failure 證據。
- 但是相同的 GPU ACPI `PEG1.PEGP._DSM` / `AE_ALREADY_EXISTS` 錯誤仍在本次 boot 重現；上一個 boot 仍符合 hard freeze/manual reset 的歷史證據。
- 判定：Athena 現在沒有正在 freeze，freeze 根因仍未證明解決；依使用者風險接受決定，可納入 R580 + CUDA 12.9 維護，但再次 freeze 就停止後續變更。HAPI hub/tunnel/stray-runner 錯誤另行處理，不作為 freeze 根因。

# Athena 反覆 kernel panic 現況（2026-08-25）

- `2026-08-25 19:52:28 +08:00` 的重開機不是排程或正常 shutdown，而是 `SCHED_SOFTIRQ`／idle scheduler 路徑中的 kernel panic；command line 的 `oops=panic panic=10` 使主機約十秒後自動重開。trace 涉及 `swapper/1`、`_nohz_idle_balance` 與 `sched_balance_update_blocked_averages`，並出現從 NX-protected page 取指的 fatal exception。
- pstore 另有 `2026-08-12` 一次與 `2026-08-18` 三次相似 panic，時間不固定；當次沒有 reboot timer、cron/at、OOM、NVIDIA Xid、thermal critical 或 MCE 證據，且系統約 97% idle。
- Athena 是 Core i9-13900K、ASUS PRIME Z790-A WIFI、BIOS `1836`、microcode `0x133`；使用者確認已關閉主機板超規設定並選用 Intel Default Settings。`0x133` 已高於 Intel 目前要求的 `0x12F`，但 panic 仍會發生。
- 同 kernel 的其他 fleet 主機沒有同類 panic；其中 Valkyrie 與 Athena 的 CPU、主機板、BIOS、microcode 和記憶體配置高度相同而未重現。因此證據偏向 Athena 單機 CPU／RAM／主機板或未捕捉的 kernel 問題，不能把 Intel Vmin Shift 或 Linux scheduler bug 任一方寫成已確診根因。
- 使用者決定先暫停此案。若日後重啟調查，先做離線 MemTest86；再用 CPU 交叉交換或保固換貨隔離 CPU，硬體測試通過後才投入 kdump 擷取完整 vmcore。不要重複建議更新 BIOS、microcode 或套用 Intel Default Settings。

# NVIDIA driver 與 CUDA 版本現況（2026-08-24）

- CUDA `12.9 Update 1` 的 Linux 完整 Toolkit driver 最低為 `R575.57.08`；五台目前均為 Ubuntu 套件的 `R580.173.02`，不把 driver 版本誤稱為 CUDA 版本。
- `mazu`、`cthulhu`、`athena`、`valkyrie`：CUDA Toolkit `13.3.1-1`，`/usr/local/cuda` 指向 `cuda-13.3`，沒有其他 Toolkit 分支。
- `zeus`：GTX 1080 Ti + R580.173.02；沒有 `/usr/local/cuda`、`nvcc` 或 CUDA Toolkit 套件，CUDA `12.9` 安裝仍待執行。

# 2026-08-24 統一結果

- 五台 Ubuntu main archive 統一為官方登記的 `https://mirror.twds.com.tw/ubuntu/`；Docker source 與 Ubuntu security source 亦保持一致。每台 `apt-get update` 與 `apt-get -s full-upgrade` 均通過，沒有待安裝或移除套件。
- 五台共同安裝的 1,635 個 APT 套件逐包比對後版本漂移為 0；各機因角色不同，安裝套件總集合不要求完全相同。
- 五台共同的 system administration baseline 為 `ethtool`、`openssh-server`、`lsof`、`dmsetup`、`lm-sensors`、`smartmontools`、`pciutils`、`dnsutils`、`tcpdump`、`rsync`、`sysstat`、`curl`、`wget`、`unzip`、`zip`；2026-08-24 由 `swear02` 逐台 live 查核，15 項全部存在且版本完全一致，`dpkg --audit=0`、`apt-get check` 通過。
- live Sheet「每台必裝」共有 53 項；明確排除 EDA 項 GTKWave 後，其餘 52 項已完成 system-wide 同步。主要版本為 GCC/G++ 12.5/13.4/14.3/15.2（default 15）、Clang 22.1.8、CMake 4.4.2、Python 3.14.4、Rust/cargo 1.93.1、uv 0.12.5、Miniconda/conda 26.5.3、Node 24.19.0/npm 11.17.0、OpenJDK 25.0.3（21.0.11 留池）、Maven 3.9.16、Docker 29.7.2/Compose 5.5.0/Buildx 0.36.1、Neovim 0.12.4。
- 69 行 system package/manual-tool manifest 在五台完全相同，SHA-256 `bd3d7a2d6f0ad0c97e3bae61dac6a81a94c1b31497e70fe8c1980fd38a48db51`；五台均通過 `dpkg --audit`、`apt-get check` 與 0-action `apt-get -s full-upgrade`。
- `/srv/shared` 在五台均指向同一 NFS backend `<shared-home>/.dvlab-shared`；目前只放計畫內的 Miniconda installer 與 shared package cache，Conda base 不自動啟用，env 仍建本機。NFS 的 numeric GID 1076 在 Cthulhu 解析成 `cursorpro`、其他主機解析成 `swear02`，不可在修正 group mapping 前用 group name 變更該樹 ownership。
- HAPI、Codex、Claude Code、OpenCode、Cursor Agent 與 agy 屬於個人／HAPI runner 使用者環境，不是 system-wide 套件。runner-user 登入 PATH 可繼續使用個人 Node 24.15.0、uv 0.11.19 與 JDK 21；clean system PATH 則解析到本輪統一版本。Zeus 唯一的 system-level AI CLI 痕跡 `/usr/local/bin/opencode` 已移除；`swear02` 仍由 `~/.local/bin/opencode` 使用相同 user-local binary。
- 五台 HAPI runner MainPID 在本輪全程未變且持續等於 state PID；未重啟 runner。舊 CMake/Neovim paths 與 alternatives 狀態已可復原地移至 `/var/backups/codex-system-tools/full-baseline-20260824/`。Zeus 舊 GCC alternatives 把 `g++` 設為 slave，現已把五台統一為 GCC master 加 `g++`/`gcov` slaves。

# Valkyrie BIOS 與 CUDA 現況

- Valkyrie 的 ASUS `PRIME Z790-A WIFI` 已完成 EZ Flash 3，讀取 DMI 確認 BIOS `1836`、日期 `04/16/2026`；因此 Cthulhu、Mazu、Athena、Valkyrie 的 BIOS 維護均已完成。Zeus 的 Supermicro BIOS 依明確決策維持不變。
- Valkyrie 目前是 Ubuntu `26.04 LTS`、kernel `7.0.0-30-generic`、RTX 4090、NVIDIA driver `580.173.02`，已安裝 CUDA Toolkit `13.3.1-1`。
- `/usr/local/cuda` 指向 `cuda-13.3`；Docker daemon 與 `swear01` HAPI runner 均 active。

# CUDA 12.x → 13.3 的性能判斷

- CUDA Toolkit 不會改變 RTX 4090 的硬體時脈、SM 數量或記憶體頻寬；對已編譯的 binary、PyTorch wheel 或自帶 CUDA userspace 的 container，單純在 host 安裝較新 Toolkit 通常不會自動變快。
- 若 workload 會用新 Toolkit 重新編譯，或實際載入新版 cuBLAS/cuDNN 等 userspace library，性能可能改變，可能變快也可能退步；不能宣稱「完全沒有差異」。
- CUDA 13.3 Update 1 對 Ada/compute capability `8.9` 不是過新，但官方 release notes 明列的 cuBLAS 性能提升主要集中在 Hopper/Blackwell，沒有 RTX 40 的普遍性能提升保證。
- 四台 Ada 主機目前以 R580 的 CUDA minor compatibility 執行 CUDA 13.3；不可把這個組合誤稱為 NVIDIA 的完整 R610/CUDA 13.3 基線。
- CUDA `12.9` freeze 只適用 Zeus 的 host Toolkit、runtime 與 workload image；四台 Ada 主機維持已驗證的 CUDA `13.3.1-1`。未來變更 driver/Toolkit 時仍需用相同 workload 做 A/B benchmark。

# 後續工作

- Zeus CUDA `12.9` 是剩餘的 GPU 維護項目；安裝前重新確認 1080 Ti 無 workload，安裝後驗證 `nvcc`、sm_61 編譯與實際執行。
- 不改變既有 HAPI runner ownership；active SQLite/WAL state 留在 host-local `/var/tmp`，不可同步回 NFS。
- Release-upgrade recovery and local-vs-centralized authentication evidence are maintained in `ubuntu-upgrade-recovery-runbook.md`。

# Valkyrie NVIDIA API mismatch（2026-09-15 實機查核）

- `dpkg` 歷史證實曾使用 R595；2026-08-22 移除 `nvidia-driver-595` / `nvidia-dkms-595` 改裝 R580，8 月 23 日改為 `580-server`。殘留 `nvidia-firmware-595-*` 不代表正在使用 R595 driver。
- APT history、unattended-upgrades log 與 systemd journal 交叉確認：9 月 12 日更新由 `apt-daily-upgrade.service` 啟動 `/usr/bin/unattended-upgrade`，不是手動 APT 操作。變更前實機啟用 `APT::Periodic::Unattended-Upgrade "1"`；timer 每日 06:00 加最多 60 分鐘隨機延遲，自動重開未啟用。
- 9 月 12 日 06:51 的套件更新將 `nvidia-driver-580-server`、DKMS 與 `libnvidia-compute-580-server` 從 `580.173.02` 升至 `580.178.04`。主機仍是 8 月 23 日的 boot，`/proc/driver/nvidia/version` 為舊版，`modinfo -F version nvidia` 與 libcuda/NVML symlink 則為新版；kernel journal 明確記錄 `NVRM: API mismatch`。
- 新版 DKMS 已在 `7.0.0-30-generic` 與 `7.0.0-31-generic` 安裝，`dpkg --audit` 無輸出，系統標示需要 restart。此故障的處置是安排重開以載入新版模組，不能將它誤判為 CuPy 必須重新安裝或缺少 R595。
- 查核時有使用者 Python 與圖形介面持有 GPU device；未執行重開或卸載模組。重開前先確認工作已保存，重開後須重新核對 loaded/on-disk driver 與 `nvidia-smi`；恢復尚未實測。

# 五台 Ubuntu 伺服器停用自動更新（2026-09-15）

- 使用者明確要求實驗室伺服器取消 Ubuntu 自動更新；已套用至 Zeus、Athena、Valkyrie、Cthulhu、Mazu。此範圍不含 Oracle、Mac、Windows 或 NAS。
- 每台 `/etc/apt/apt.conf.d/20auto-upgrades` 的 `APT::Periodic::Update-Package-Lists` 與 `APT::Periodic::Unattended-Upgrade` 均改為 `"0"`，並執行 `systemctl disable --now apt-daily.timer apt-daily-upgrade.timer`。
- Athena、Valkyrie、Cthulhu、Mazu 的 Snap 執行 `snap refresh --hold`，以全域 `hold: forever` 停止自動 refresh；Zeus 沒有 Snap。此模式保留手動 `snap refresh`，APT 也保留手動更新，安全更新改由人工安排。
- 五台均回查有效 APT 值為 0、兩 timer 為 `disabled/inactive`、APT daily services 為 `inactive`、`dpkg --audit` 無輸出；四台 Snap 都回報 `hold: forever`。未重開主機或更換 driver；Valkyrie 原有 API mismatch 尚待重開後驗證。
- 每台原設定與 timer/Snap 狀態已備份於本機 `/var/backups/disable-auto-updates-20260915-*`。還原時取該機備份的 `20auto-upgrades`，再 `systemctl enable --now apt-daily.timer apt-daily-upgrade.timer`；四台 Snap 用 `snap refresh --unhold`。恢復自動更新須有新的使用者授權。

# 五台 driver mismatch 重開處理（2026-09-15，未完成）

- 使用者授權檢查五台主機，對 driver mismatch 主機重開。實測五台 `nvidia-smi` 都報 `Driver/library version mismatch`：loaded module 均 `580.173.02`，userspace 為 `580.178.04`。
- Mazu 使用 Ubuntu 預編譯 `linux-modules-nvidia-580-server-*`，沒有 DKMS 記錄；目前 `7.0.0-30` 的磁碟 module 是舊版，但 GRUB 預設的 `7.0.0-31` module 已是 `580.178.04`。不能僅憑目前核心的 `modinfo` 或空 DKMS 清單判成缺少新版 driver。
- 已對 Athena、Valkyrie、Cthulhu 發出正常 `sudo systemctl reboot`，指令回傳成功。之後從 Zeus 與 Mazu 均無法連回三台；ARP/ping/SSH 未恢復，一次已登記 MAC 的 Wake-on-LAN 亦尚未恢復。這只證明遠端不可達，不證明關機、kernel panic 或硬體故障；已請使用者查看電源燈與主控台畫面。
- Zeus（本對話所在主機、NIS master）與 Mazu（HAPI Hub）保持運作，尚未重開，以保留管理入口。不得將這次工作標成已全部完成；三台需先恢復並通過 boot time、loaded/on-disk driver、`nvidia-smi`、NIS/NFS/HAPI 與自動更新停用設定檢查，再安排剩餘兩台。

# Snap AppArmor 與 NFS PATH 開機死鎖（2026-09-15 13:50 後查核）

- 後續使用者再次授權重開剩餘兩台，Mazu / Zeus 已分別排定 13:24:28 / 13:24:29 正常 shutdown reboot。13:50 查核 Zeus、Athena、Mazu 都已恢復，kernel `7.0.0-31-generic`、loaded NVIDIA `580.178.04` 與 `nvidia-smi` 正常。Valkyrie、Cthulhu 仍無法 SSH；不得推定它們已恢復或已確診同一根因。
- 使用者從 Recovery mask `snapd.apparmor.service` 後，Athena 與 Mazu 恢復正常開機；兩台目前都是 `/etc/systemd/system/snapd.apparmor.service -> /dev/null`。Zeus 沒有安裝 Snap，也沒有此 service。
- 已找到兩台失敗 boot 的共同直接證據：journal 記錄 `apps.automount: Got automount request for /apps, triggered by ... (snapd-apparmor)`；`apparmor.service` 已正常完成，`snapd.apparmor.service` 只出現 Starting，沒有 Finished。Athena 是 13:05:50、Mazu 是 13:25:46。
- 兩台 `/etc/environment` 都把 `/apps/bin` 放在 PATH 最前；Snap unit 透過 `EnvironmentFile=-/etc/environment` 載入它，且 `DefaultDependencies=no`、`Before=sysinit.target`。`/apps` 是 NFS `x-systemd.automount`，其 mount 等 `network-online.target`；NetworkManager 與 wait-online 又排在 `sysinit.target` 之後。Snap 觸發掛載 → 等網路 → 等 sysinit → 等 Snap，形成執行期等待環。
- Snap `2.76.3` 官方 `cmd/snapd-apparmor/main.go` 在載入 profiles 前呼叫 `systemd.IsContainer()`；套件函式文件說明它呼叫 `systemd-detect-virt --quiet --container`。這與全域 PATH 觸發 NFS 的機制一致；尚未對正在卡住的程序取得 syscall trace，故不把確切的首次檔案存取寫成已實測。
- `/etc/environment` 與 fstab 的 mtime 分別為 Athena 9 月 14 日、Mazu 9 月 11–12 日；Athena 的 distro snapd / AppArmor 套件安裝紀錄為 8 月升級。沒有證據把死鎖歸因於今天的 `snap refresh --hold`；hold 只抑制更新，並不取消早期 AppArmor 服務。
- 最小根因修正方向：早期 system service 使用僅本機路徑的 PATH，`/apps/bin` 留在使用者登入環境；修正與驗證後才恢復 Snap AppArmor。這輪僅查核，未改 PATH、unmask 或再次 reboot。mask 會跨重開持續生效，不是自動解除的一次性措施；不要將整體 `apparmor.service` 停掉。
- 驗證方法注意：`snapd-apparmor --help` 不受支援，回 exit 1 並在 validateArgs 提前結束；本次非 root strace 只看到 re-exec 到 snapd snap，不能用它宣稱已驗證 `start` 的 PATH 存取或 profile loading。

# 三台線上主機 PATH 根因修正完成（2026-09-15 14:00）

- 使用者要求先處理 Zeus、Athena、Mazu。Zeus 的 APT history 證實 6 月 22 日由管理帳號執行 `apt-get purge -y snapd`；記錄沒有保留移除動機。三台 `/apps` 的 NFS4 + systemd automount 設定完全相同；`/home` 才是 Zeus/Athena 用 fstab、Mazu 用 autofs 的既有差異，此次未變更掛載架構。
- 三台僅從 `/etc/environment` 的 PATH 移除 `/apps/bin:`；共用工具繼續由 `/etc/profile.d/zz-dvlab-apps-path.sh` 加入登入 shell。同步刪掉該檔與 `Z20-dvlab-apps.sh` 中「/etc/environment 提供共享 PATH」的過時註解，其他行為不變。三份檔案的 SHA-256 在三台逐檔相同。
- 先修 Athena、unmask 並啟動 `snapd.apparmor.service`，再實際正常重開；boot time 為 13:57:10，Snap AppArmor 13:57:19 Finished，同秒 sysinit 完成，13:57:20 NetworkManager 啟動。通過後才修 Mazu / Zeus。
- Mazu 首次正常 reboot 被 block inhibitor 拒絕；檢查 inhibitor 已無 shutdown block 後，只重試一次相同 `systemctl reboot`，沒有 force。Mazu boot time 13:59:37，Snap AppArmor 與 sysinit 13:59:54 完成，NetworkManager 13:59:55 啟動。兩台 service 均 `enabled`、`active/exited`、Result success；原本的 recovery mask 已解除，Snap AppArmor 保護已恢復。
- Zeus 沒有 Snap，套用相同 PATH 修正後驗證新登入環境即可，本輪沒有再次重開 Zeus。三台 NVIDIA `580.178.04` 正常；Athena / Mazu failed units 為空，NetworkManager、Docker、ypbind、SSSD、HAPI Runner 正常；Zeus NIS master、Docker、Runner 正常。Mazu Hub/Tunnel 正常，HAPI LAN/public HTTP 都為 200。
- 三台新 SSH login shell 均含 `/apps/bin`，可讀 `/apps/etc/env.sh`，實際共享來源是 NAS 的 `/volume1/apps` NFS4。系統 `/etc/environment` 不含網路路徑。APT 有效 Unattended-Upgrade 值仍為 0，兩 timer disabled/inactive；Athena / Mazu Snap 仍 `hold: forever`。
- 原檔與 mask link 備份保留於各主機 `/var/backups/snap-nfs-path-20260915-*`。復原時不可直接把網路 PATH 放回 `/etc/environment`，否則會恢復已實測的 boot deadlock。Valkyrie / Cthulhu 尚未套用此根因修正，需恢復連線後再依使用者指示處理。
- 後續維護原則：開機基礎服務只搜尋本機工具，共享工具由登入環境提供；服務若確實需要 NFS，應明確安排網路及 mount dependency。不要為了表面一致而替 Zeus 安裝 Snap，或移除其他主機仍在使用的 Snap 應用；維持逐台修改、逐台重開驗證。

# 五台工具與載入方法查核（2026-09-15 14:20）

- 使用者要求調查統一工具與方法。本輪只讀查核五台，未變更系統設定或再次重開。Valkyrie / Cthulhu 已恢復 SSH，五台 `nvidia-smi` 均成功、driver `580.178.04`。兩台 `/etc/environment` 仍含 `/apps/bin`，尚未套用前三台的根因修正；Valkyrie Snap AppArmor enabled/active，Cthulhu masked/inactive，後者另有兩個 Snap CUPS failed units。能登入不代表開機根因已修復。
- 五台 `/apps` fstab 行逐字一致，使用同一 NAS export、NFS4 與 `x-systemd.automount`。七個 `/apps/bin` 入口、`/apps/etc/env.sh`、`/apps/eda/docker-shell` 的 SHA-256 均逐檔一致。`/home` 則仍混用 fstab 與 Mazu autofs，屬另一個須安排切換的議題。
- 完整 installed dpkg inventory：Zeus 2747、Athena 2986、Valkyrie 3176、Cthulhu 3283、Mazu 3031；交集 1735 項，其中 1733 項版本一致。僅 `libinput-bin` / `libinput10:amd64` 有版本差異：Zeus / Valkyrie 為 `1.31.1-1ubuntu1.2`，其餘三台為 `1.31.1-1ubuntu1.1`。交集不代表五台安裝集合相同。
- 明確指定純本機 system PATH 後，五台實際執行版本一致：GCC 15.2.0、Clang 22.1.8、CMake 4.4.2、Python 3.14.4、Node 24.19.0、npm 11.17.0、uv 0.12.5、Docker 29.7.2。管理帳號 login shell 都能找到 `/apps/bin/vcs`，也都被個人 Node / uv 與 Miniconda Python 覆蓋；不能用 login shell 的解析結果代表 system baseline。
- bare SSH 指令在已修三台的 PATH 不含 `/apps/bin`，未修兩台仍含它；login shell 五台皆含共享入口。批次作業須明確 source 專案工具版本，不能依賴 SSH 是否觸發登入設定，更不能把 NFS PATH 放回早期 service 環境來補救。
- 共享 EDA 入口方法仍有差異：VCS、Verdi、Design Compiler、JasperGold 要求先 source 選版；Innovus 與 SpyGlass 入口仍自動載入舊 CIC 腳本。應沿用現有 `/apps/eda` 手動選版機制收斂，不另加平行的工具管理系統。
- 五台都有 `dvlab-eda:rocky8-20260915-r5`；Zeus / Athena / Mazu image ID 為 `68d8efc404f5…`，Valkyrie / Cthulhu 為 `fa24ff933248…`。進一步比較 Zeus / Valkyrie 的六個 RootFS layer digest、Created、Architecture、Cmd 與 Entrypoint 均相同；不可由不同 image ID 直接宣稱工具內容不同，亦不可只憑相同 tag 宣稱完整 image 相同。
- 四台 Snap 套件版本相同，但應用清單及 revision 不完全相同；Zeus 無 Snap。APT 自動更新仍關閉，四台 Snap 都 hold forever。建議統一套件來源、專案選版與人工維護規則，保留硬體及主機角色差異；此次未安裝 Zeus 尚缺的 CUDA 12.9 Toolkit，也未更新 Snap 或 libinput。

# 在線統一設定完成範圍（2026-09-15，明確禁止重開）

- 使用者授權處理上述統一事項，但明確禁止重開。Valkyrie / Cthulhu 已補上與前三台逐檔相同的 `/etc/environment`、`Z20-dvlab-apps.sh`、`zz-dvlab-apps-path.sh`；各自備份在 `/var/backups/fleet-unify-20260915-*`。Cthulhu 已 unmask / start Snap AppArmor，其兩個 CUPS failed units 的 journal 明確記錄 `missing profile snap-update-ns.cups`；AppArmor 恢復後重啟這兩個原本已失敗的服務，均 active/running。
- 三個共享入口 `/apps/bin/innovus`、`spyglass`、`sg_shell` 不再自動 source CIC，改為要求使用者先載入環境。SpyGlass 以已 export 的 `SPYGLASS_HOME` 執行；Innovus 的新版入口支援已選定的 `EDA_CADENCE_DDI`。但新版 DDI 目錄仍是 `.adopting-25.10.000`，SpyGlass 新版安裝也未完成，所以現階段使用者仍明確 source 舊 CIC 固定版本腳本，再從 PATH 執行原安裝工具。未移轉或發布未完成的 EDA 安裝。
- 已更新 `/apps/eda/README.md` 的手動選版與批次 SSH 用法，新增 `/apps/etc/MAINTENANCE.md` 作為五台共同維護規則。舊 CIC 並非多版本選擇器，不能在末尾加版本號期待切換。共享入口及 README 原檔備份位於 Zeus `/var/backups/fleet-tools-20260915-*`。
- Athena / Cthulhu / Mazu 的 `libinput-bin` 與 `libinput10:amd64` 精確升到 `1.31.1-1ubuntu1.2`，使用 APT 驗證來源下載的兩個 deb；每台先模擬確認僅 2 upgraded / 0 new / 0 remove，使用 `NEEDRESTART_MODE=l` 並禁止移除，沒有 full-upgrade、autoremove、driver 或 kernel 更新。完成後重新取得五台完整 dpkg inventory：共同 1735 項版本差異為 0。
- 五台 boot ID 前後一致、無 scheduled shutdown；NVIDIA `580.178.04`、Docker、HAPI runner、NFS、新登入共享 PATH 與 dpkg audit 均通過，failed units 全空。Zeus 網路由 active systemd-networkd 管理，NetworkManager inactive 不代表網路故障；NIS master active。Mazu Hub / Tunnel active。APT timers disabled/inactive，四台 Snap hold forever 且 AppArmor enabled/active。
- 三個入口通過 `bash -n`，最小可重跑 regression check 保留在 `/apps/eda/.admin/rollout-20260915/validation/check-manual-entrypoints.py`，驗證未選版時拒絕執行與選版後參數完整傳遞；五台乾淨環境都實測 source 既有 Innovus / SpyGlass 及 VCS 2025.06 後解析到可執行檔。此次未進行完整 EDA 授權／設計工作負載測試。
- `/home` 的 fstab / autofs 差異保留，沒有卸載、重掛、停止 autofs 或預埋未測試的下次開機設定；共同文件已寫入維護時段切換條件。這項與未完成的 EDA 新目錄移轉仍是待辦，不能宣稱所有架構已完全一致。Valkyrie / Cthulhu 新設定僅在線驗證，沒有重新開機驗證。

# Mazu home 切換預檢（2026-09-15）

- 使用者要求研究並處理 home 掛載切換，接受暫時中斷，但前述禁止重開仍有效。實機除了 HAPI Hub / Tunnel / Runner 外，尚有使用者的 `abc` 計算、tmux 與遠端開發程序，不能把停止服務等同於可無損暫停所有計算。
- autofs dumpmaps 確認唯一有效掛載為 `/home`，wildcard 指向 NAS 的同一 nfs-home export；根 mount 與 `/home` 都是 shared propagation。官方與本機 automount 手冊說明，強制 unlink 會讓既有 cwd 的 getcwd / proc 路徑失效，不能用 lazy unmount 宣稱無損切換。
- 已在 Mazu 的獨立 `/mnt` 暫存掛載測試完整 NAS home export，使用 `nfs4,rw,hard,vers=4.1,rsize=32768,wsize=32768`。五個當下使用中的家目錄在新舊路徑的 device/inode/UID/GID/mode 全部相同，確認不需要搬資料或重設 owner；測試後正常卸載並移除暫存 mount point。
- `/var/backups/mazu-home-cutover-azp2cjmf` 保存現有 fstab / auto.master / auto.nfs、boot ID / mountinfo，以及 proposed fstab / auto.master。提案改以 fstab 的 NFS4 + `_netdev,nofail,x-systemd.automount` 管理完整 `/home`，移除舊 master map 的 `/home` 行。`findmnt --verify` 為 0 parse errors / 0 errors，僅既有 swapfile regular-file warning；沒有寫入正式設定，沒有停 autofs 或終止使用者程序。
- 正式切換仍需先處理正在執行、無法保證保存進度的使用者工作，再從本機磁碟上的 root 維護程序停止使用 home 的服務、正常卸載、套用與啟動新 mount，失敗則回復備份及 autofs。不可將預檢成功寫成切換完成。

# Valkyrie recovery 要求 root 密碼的實際原因（2026-09-15）

- 五台 `passwd -S root` 實測：Valkyrie 為 `P`（有可用密碼），最後密碼變更日期 `2026-05-04`；Zeus / Athena / Cthulhu / Mazu 都是 `L`（密碼鎖定）。只查狀態，不輸出 password hash；沒有改密碼、鎖帳號或其他認證設定。不能套用「Ubuntu 預設鎖 root」就宣稱 Valkyrie 也鎖住。
- 五台 friendly-recovery `0.2.42build1`、systemd `259.5-0ubuntu3.4`、util-linux `2.41.3-3ubuntu2.2` 一致；recovery-menu、options/root、systemd-sulogin-shell、sulogin 的 SHA-256 各自一致，rescue / emergency service 也無不同 drop-in。Recovery Menu 的 root 選項實際就是 `/sbin/sulogin`，所以僅憑密碼提示不能判定它進入不同 emergency mode。
- 使用隔離 PTY 執行實機 `sulogin -t 1`（不送任何密碼或 Enter，備用 SUSHELL 指向 `/usr/bin/true`）重現：Athena 顯示 `Press Enter for system maintenance`，Valkyrie 顯示 `Enter root password for system maintenance`；皆逾時退出，沒有啟動互動 root shell。這直接證明相同程式會依帳號狀態顯示不同提示，不依賴上游手冊對其他發行版的預設行為推測。
- Valkyrie 當前 kernel cmdline 仍有 `recovery nomodeset dis_ucode_ldr`；journal 顯示本次走 friendly-recovery.target / service，之後 recovery menu resume 回 graphical.target。沒有證據支持使用者貼文中「Valkyrie 一定走 emergency、其他主機走 recovery」的斷言；較早未保留的 console 畫面亦不能由當前日誌完全還原。
- 5 月 4 日的 passwd journal 查無紀錄，不能確定密碼設定者或原因。root 密碼狀態不同與 Snap/NFS PATH 開機等待環是不同問題；本輪僅調查，沒有重開、改 root 認證或繼續 home 切換。

# Mazu home 切換與 Valkyrie root 一致化完成（2026-09-15）

- 使用者明確同意結束 Mazu 上的使用者工作後切換 home，並要求 Valkyrie 設定與其餘四台一致；禁止重開仍有效。Valkyrie 已 `passwd -l root`，五台 `passwd -S root` 都為 L；沒有刪除密碼或設空密碼，sudo 仍正常。原始 shadow 與狀態備份在 Valkyrie root-only `/var/backups/root-login-align-20260915-*`。隔離 PTY 的 `sulogin -t 1` 實測已變成 Press Enter 提示，測試不送密碼或 Enter，備用 shell 為 true。
- Mazu 從本機 `/var/backups/mazu-home-cutover-azp2cjmf/cutover.sh` 的獨立 root systemd oneshot 執行，暫設 `/run/nologin` 防止切換中登入，停止 HAPI 等 home 使用者服務，依授權結束登入工作，再正常停止 autofs、套用 proposed fstab / auto.master、啟動 home.automount。沒有 force/lazy unmount，沒有 reboot。
- 首次 15:08 的切換因驗證程式誤判而自動還原：`findmnt -rn -T <home-user> -o SOURCE` 在 systemd automount 上回傳 autofs 與 NFS 兩層，直接和單一 export 字串比較必定失敗。已實測原設定與 autofs / HAPI 恢復，修正為 `findmnt -rn -t nfs4 -T <home-user> -o SOURCE` 並先在既有 apps automount 驗證選型，再進行第二次切換。這是驗證程式問題，不是 NAS 掛載失敗。
- 第二次 15:09:32–15:09:42 成功：fstab 管理完整 NAS nfs-home export 到 `/home`，NFS4.1、hard、32768 rsize/wsize、_netdev、nofail、x-systemd.automount。`home.mount` / `home.automount` 由 `/run/systemd/generator` 產生且 active；autofs.service disabled/inactive，原 `/etc/auto.nfs` 保留備份用途但不再生效。五台 home 現都由 fstab 管理，但其餘四台仍保留原掛載選項，不能宣稱所有選項完全一致。
- 五個當時使用者各自執行暫存檔 write/read/fsync 成功且測試檔已自動刪除；SSH、Docker、ypbind、SSSD、NVIDIA 580.178.04 正常。HAPI Hub / Runner / Tunnel active，LAN/public HTTP 200；failed units 清單為空，維護 nologin 已移除。五台 boot ID 均與切換前相同，沒有實際重開驗證。已結束的使用者計算與開發工作需自行重啟。
- `/apps/etc/MAINTENANCE.md` 已更新現況、授權範圍、root 密碼鎖定與切換方式；備份保留在 Zeus `/var/backups/MAINTENANCE.pre-home-cutover-20260915.md`。先前 home 切換待辦已完成；EDA 未完成安裝移轉不屬此輪，仍保留。
