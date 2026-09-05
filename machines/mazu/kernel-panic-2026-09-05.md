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
- EFI pstore 記錄時間是 `06:18:09`，與日誌突然停止相差約 24 秒。
- `systemd-pstore` 在下一次開機封存兩組壓縮 `dmesg-efi_pstore`，合併後各約 34 KiB、36 KiB，每組各有 23 個 EFI fragments。

# 已確認判斷

- `/sys/module/printk/parameters/always_kmsg_dump=N`；依本機 `systemd-pstore(8)`，正常 shutdown、reboot、halt 只有在該參數開啟時才會寫入 pstore。這批記錄因此證明前一個 kernel 走過 crash／panic 類型的 fatal path，不是正常關機。
- 關機前 HAPI hub 仍每數秒正常完成請求，最後成功後日誌直接中斷，沒有使用者服務停止或 graceful shutdown 序列。
- 使用者沒有 crontab、`at` 工作或 user timer／unit 中的 `poweroff`、`shutdown -h`、`halt` 指令。
- `panic_on_oom=0`、`panic_on_oops=0`、hard／soft lockup panic 均為 `0`；這些是下一次開機讀值，不能代替事故當下的 root journal，但沒有發現刻意設定的自動 panic policy。
- `issue25-v5-downstream-thermal-guard.service` 是共享 home 留下的遠端 Issue25 unit；Mazu 上的本機腳本不存在，開機後只會以 `203/EXEC` 失敗，不能是本次主動關機來源。

# 過熱證據與界線

- 重開後在 CPAchecker Java 與備份 `tar/gzip` 工作同時執行時，Core i9-14900K package 連續量到 `93–96°C`；硬體規格的 Tjunction／最高操作溫度是 `100°C`。
- 本次 boot 已累積每個 logical CPU 約 `774–775` 次 package thermal throttling、約 `3.9 s` throttle time，部分 core 另有數百次 core throttling。這證明目前散熱餘裕不足，但不能單靠重開後的讀值把事故 trace 判成 thermal shutdown。
- GPU 當時重開後的即時讀值約 `35°C`、閒置；這也不能回推事故當下 GPU 狀況。

# 尚缺的唯一關鍵證據

- 合併後的兩份 pstore `dmesg.txt` 是 `root:root 0640`，現有 runner 帳號和 sudo policy 都不能讀。必須由管理者擷取其中的 `Kernel panic`、`BUG/Oops`、`RIP`、`Call Trace`、`NMI watchdog`、`MCE/Hardware Error`、`thermal`、`NVRM/Xid` 行，才能把根因定到 scheduler、driver、CPU/RAM/主機板或過熱。
- 同 fleet 的 Athena 曾在相同 Ubuntu `26.04`、kernel `7.0.0-30-generic`、NVIDIA `580.173.02` 上發生 `SCHED_SOFTIRQ`／idle scheduler panic；見 `machines/hapi-fleet/ubuntu-gpu-maintenance-plan.md`。在讀到 Mazu stack trace 前只可列為比對候選，不可宣告同一根因。
