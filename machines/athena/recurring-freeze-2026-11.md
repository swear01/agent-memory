---
title: Athena 2026-10/11 反覆 freeze 與 BERT 硬體錯誤
scope: machines/athena
machine: athena
status: investigating
confidence: medium
created: 2026-11-24
updated: 2026-11-24
tags: [kernel, freeze, bert, hardware, raptor-lake]
---

# Athena freeze：分開辨認 preempt_count、硬體與 watchdog

- 用 `su_athena` 查 previous-boot journal、pstore、`/var/crash`。非正常結束前曾有 `softirq: huh, entered softirq 7 SCHED ... preempt_count`，與 idle scheduler（`_nohz_idle_balance`）panic 指紋相近。2026-11-24 已驗整機卡死時 `oops=panic panic=10`、`kexec_crash_loaded=1` 也沒有觸發 dump，只能由現場 reset。新開機的 `BERT: [Hardware Error]: Skipped 1 error records` 是韌體證據，未印完整內容不代表沒有硬體錯誤。保留 `/sys/firmware/acpi/tables/data/BERT`，以同級主機和不同負載交叉驗。
- 全核 stress 曾過，Innovus 的部分核心高頻負載仍復發；不能用送修歸還或滿載燒機當作修好。歷史可辨識 CPU 的異常集中 P-Core 0 的 CPU 0–1，`softirq: huh` 本身沒有 caller CPU。單機 CPU／RAM／主機板仍待隔離；Raptor Lake Vmin 只是候選，復發也不能直接判 Core 1 同樣退化。既定隔離順序為離線 MemTest86 再以授權硬體交叉驗證或保固，避免切換現役主機無法保存的工作。
- iTCO watchdog：無 ACPI WDAT，Ubuntu 自動載入 iTCO 可能被 blacklist。當時明確啟用 `iTCO_wdt` 並寫 `/etc/modules-load.d/iTCO_wdt.conf`、`/etc/systemd/system.conf.d/watchdog.conf`，`RuntimeWatchdogSec=5min`／`RebootWatchdogSec=10min`。避免 30 秒在 kdump capture kernel 不餵 watchdog 時提前 reset。用 `systemctl show -p WatchdogDevice`、`wdctl` 確認，真 freeze reset 與 reboot 接手未驗。回復只移除當次兩個設定，保留後續改動及檢查 watchdog ownership。
- CPU 0 是 BSP；下線 CPU 1 曾使 Bun `os.cpus()[0].times` 拋 `ERR_SYSTEM_ERROR: Failed to get CPU information` 並讓 HAPI restart loop。維持 CPU 0–1 在線，由 `/etc/systemd/system.conf.d/pcore0-quarantine.conf` 設 `CPUAffinity=2-31`，`init.scope`、`system.slice`、`user.slice`、`machine.slice` 的 `AllowedCPUs=2-31` 讓 userspace 避開 P-Core 0。 `athena-pcore0-quarantine.service` 及 `/usr/local/sbin/athena-pcore0-quarantine.sh` 將 CPU 0–1 的 `scaling_max_freq` 設 3.0 GHz、EPP power，`default_smp_affinity=fffffffc` 及可搬 IRQ 避開。驗 CPU frequency／affinity、HAPI 及負載下是否再現。短時無 softirq 不證明已修好。
- 回復 affine/quarantine 要先確認原檔、drop-in 與 slice 沒有後來的有效調整，再依收據停該 service、撤專用檔案和四個 unit 的 `AllowedCPUs` 後 daemon-reload。對於共享 NFS 的 `user@1075` 抖動和 `fusermount3` denials，另查 Zeus-only user units 的 `ConditionHost`、殘留 firmware notifier 和 `swear02` linger；這些當時修好日誌雜訊，沒有證明能消除 freeze。

[各 boot 對照、watchdog 閘門、備份和逐項回復命令](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/athena/recurring-freeze-2026-11.md)
