---
title: Mazu 2026-09-05 kernel crash 與過熱風險
scope: machines/mazu
machine: mazu
status: investigating
confidence: high
created: 2026-09-05
updated: 2026-09-05
tags:
  - kernel-panic
  - pstore
  - thermal
  - ubuntu
---

# 已確認時間線

- 前一個 boot 的最後一筆可見應用日誌是 `2026-09-05 06:17:44.894 +08:00`；下一個 boot 從 `15:54:13` 開始。
- `06:17:00.614`，CPU 8 的 idle thread `swapper/8` 先出現 `BUG: scheduling while atomic`，preempt count 是 `0x00000002`。
- `06:18:09.796`，同一 CPU 在 timer interrupt 的 `SCHED_SOFTIRQ` 路徑發生 fatal page fault 並 panic；這比最後一筆應用日誌晚約 25 秒。
- `systemd-pstore` 在下一次開機封存 `Oops#1` 與 `Panic#2` 兩組記錄。兩者有相同時間戳、CPU、CR2 和 trace；這是同一事故的 Oops／panic 階段，不是兩次獨立 crash。

# 遠端復原邊界

- 當時 Mazu 的 LAN 鄰居解析、SSH、HAPI Hub 與公開 HAPI 入口均不可用。這只能證明 OS／網路未運作，不能單靠網路狀態區分關機、panic 或開機卡住。
- Zeus 已向 Mazu 傳送兩輪 Wake-on-LAN magic packet（UDP 9 與 7），等待約六分鐘後仍無 LAN／SSH／Hub 回應。這證明現有 WoL 路徑不能將已處於 kernel panic／硬卡死的運作中主機當作硬體 Reset；不代表 Mazu 從正常關機狀態的 WoL 已被否定。
- 使用者手動重開後，`hapi-runner.service`、`hapi-hub.service` 與 `hapi-tunnel.service` 均自動回到 `active/running`，本機與公開 HAPI HTTP 均為 `200`。
- 當前 `kernel.panic=0`、`kernel.panic_on_oops=0`，且沒有 `/dev/watchdog*`。因此同類 panic 不會自動重開；最小軟體改善是設定有限的 `kernel.panic` 秒數，完整的硬卡死復原則需硬體 watchdog、BMC／IPMI 或可遠端控制的 PDU。這些改動尚未施作。

# 已確認判斷

- `/sys/module/printk/parameters/always_kmsg_dump=N`；依本機 `systemd-pstore(8)`，正常 shutdown、reboot、halt 只有在該參數開啟時才會寫入 pstore。這批記錄因此證明前一個 kernel 走過 crash／panic 類型的 fatal path，不是正常關機。
- fatal trace 是 `kernel tried to execute NX-protected page`、`supervisor instruction fetch in kernel mode`，最後為 `Kernel panic - not syncing: Fatal exception in interrupt`。
- faulting `RIP=ffffd32e8036cea8` 位於當下 kernel stack 範圍，且只比 `RSP=ffffd32e8036ce88` 高 `0x20`；CPU 嘗試把 stack data 當指令執行。這證明直接故障型態是 kernel 控制流或 stack 被破壞，不是正常函式內的一般 NULL dereference。
- trace 的有效外層是 `sched_balance_update_blocked_averages` → `sched_balance_softirq` → timer IRQ；當時 CPU 正在 `cpuidle_enter_state`。`preferred_group_nid` 與 `timerqueue_add` 是在已損壞 stack 上展開出的內層 frame，不能單獨認定為根因函式。
- 關機前 HAPI hub 仍每數秒正常完成請求，最後成功後日誌直接中斷，沒有使用者服務停止或 graceful shutdown 序列。
- 使用者沒有 crontab、`at` 工作或 user timer／unit 中的 `poweroff`、`shutdown -h`、`halt` 指令。
- 完整 previous-boot journal 與 pstore 都沒有 MCE、EDAC error、PCIe AER error、NVIDIA Xid、thermal critical、watchdog、OOM、NVMe 或 filesystem failure。kernel taint `P/O` 只表示 proprietary/out-of-tree NVIDIA modules 曾載入；NVIDIA 不在實際 call trace 中。
- sysstat 在 `06:00`／`06:10` 顯示整機約 `96.6%`／`96.5%` idle、load average 約 `1.1`、沒有 blocked task，約 `121 GiB` memory available。事故不是全機高負載或記憶體耗盡時發生。
- `issue25-v5-downstream-thermal-guard.service` 是共享 home 留下的遠端 Issue25 unit；Mazu 上的本機腳本不存在，開機後只會以 `203/EXEC` 失敗，不能是本次主動關機來源。

# 過熱證據與界線

- 重開後在 CPAchecker Java 與備份 `tar/gzip` 工作同時執行時，Core i9-14900K package 連續量到 `93–96°C`；硬體規格的 Tjunction／最高操作溫度是 `100°C`。
- 本次 boot 已累積每個 logical CPU 約 `774–775` 次 package thermal throttling、約 `3.9 s` throttle time，部分 core 另有數百次 core throttling。這證明目前散熱餘裕不足，但不能單靠重開後的讀值把事故 trace 判成 thermal shutdown。
- 事故前的 journal 沒有 thermal-critical 訊息，且 sar 顯示整機大多 idle；因此過熱是需要另行處理的現況，但不是這次 panic 的首要解釋。
- GPU 當時重開後的即時讀值約 `35°C`、閒置；這也不能回推事故當下 GPU 狀況。

# 根因界線與下一步

- Mazu 的指紋與 Athena 先前四次事故同屬 `SCHED_SOFTIRQ`／idle scheduler、`sched_balance_update_blocked_averages`、NX stack execution。現在可判定為同一故障類別；兩台都使用 Ubuntu `26.04`、kernel `7.0.0-30-generic`、Raptor Lake i9、Z790、microcode `0x133`。
- 這使 shared kernel/platform fault 明顯比 HAPI、CPAchecker、NVIDIA 或單純過熱更可疑，但 pstore 尚不能二選一：可能是 kernel scheduler／preemption race，也可能是 CPU、RAM 或主機板造成 silent control-flow corruption。沒有 MCE 不足以排除後者。
- Mazu 的 `USE_KDUMP=0` 且 `kexec_crash_loaded=0`，所以本次沒有 vmcore。Ubuntu repository 已有 candidate `7.0.0-31.31`（含 upstream 7.0.13/7.0.14），但 changelog 沒有可直接對應此 trace 的修正，不可先宣稱新版已修好。
- 最小隔離順序：先用新版 kernel 做 A/B；若仍重現，改用既有 `6.8.0-138` 做 A/B；再做離線 MemTest86 與 CPU 交叉交換／保固隔離。若要取得能定到 instruction/資料結構的證據，必須先啟用並驗證 kdump，確認 `kexec_crash_loaded=1`。
