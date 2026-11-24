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

# Athena 反覆 freeze（2026-11-24 查核）

延續 [[machines/hapi-fleet/ubuntu-gpu-maintenance-plan]] 的「Athena 反覆 kernel panic」與 [[machines/mazu/kernel-panic-2026-09-05]]。

- 一般帳號 `athena` 沒有 sudo；看 system journal、pstore 用 `su_athena`。
- 2026-10-01 起 16 個 boot 中，**每個沒有 `Journal stopped`（非正常結束）的 boot 都出現 `softirq: huh, entered softirq 7 SCHED ... preempt_count 00000101, exited with 00000102?`**（每 boot 1–17 次）；正常 reboot 的 boot 幾乎沒有。這是 8 月 panic trace（SCHED_SOFTIRQ／`_nohz_idle_balance`）同一路徑的 preempt count 洩漏。
- 現在的死法是整機 freeze，不是 panic：`oops=panic panic=10` 與 kdump（`kexec_crash_loaded=1`）都沒觸發，pstore、`/var/crash` 皆空。11-23 23:55 死後停了 13 小時；11-24 14:22 死、14:33 回來。使用者確認兩次都是手動按 reset 才恢復，主機不會自己重開。
- 11-24 的兩次開機都有 `BERT: [Hardware Error]: Skipped 1 error records`，代表韌體記下前一次 boot 的硬體錯誤（記錄 >1 KiB 所以 kernel 沒印內容）；更早的 boot 沒有 BERT 記錄。
- 對照：Valkyrie、Cthulhu（同 i9-13900K、microcode `0x133`、kernel `7.0.0-43`）與 Mazu（14900K）自 10-01 起 `softirq: huh` 與 BERT 都是 0。證據指向 Athena 單機 CPU／RAM／主機板，Raptor Lake 已發生的 Vmin 退化最可疑（microcode 只能防止惡化，不會修復），但尚未經硬體交換確診。
- 送修期（`10/19–11/11`，Boot `-9..-6`）主機板網卡 MAC（`a0:36:bc:ad:e0:6f`）、CPU、4 條記憶體與 BIOS 均未變；期間曾跑全核 `stress-ng` 約 36 小時（壓到觸發 OOM killer，但 0 次 `softirq: huh`、0 次死當），回機後一跑 `innovus`（15%–47% 部分核心負載）數分鐘即噴 `softirq: huh` 並 freeze，吻合兩顆 `5800 MHz` TVB 偏好大核（`Core 0` = `CPU 0,1`、`Core 1` = `CPU 2,3`）在單／少核高頻 Boost 與進出 idle 時的 Vmin 不穩。
- 歷史所有印出 CPU 編號的異常（8 月 `swapper/1` panic、10 月兩次 `swapper/0` `scheduling while atomic`、11/15 `abc_v25` segfault `likely on CPU 0`）皆落在實體 `Core 0`（`CPU 0`/`1`），但 `softirq: huh` 無 caller CPU 欄位；`/sys/firmware/acpi/tables/data/BERT` 含 3,424 bytes Intel Raptor Lake CPU/PMC CrashLog（`0x41037305` / `0x11037101` / `0x17037101`），其中 2 個區塊有非零值。x86_64 Linux 無 `cpu0/online`，無法將 `CPU 0` offline。
- 雜訊非根因：`fusermount3` AppArmor DENIED 原由 `user@1075` 每分鐘啟停觸發（已於下方修正）；`earlyoom` 顯示記憶體 99% 空閒，無 OOM、Xid、thermal。
- 下一步沿用既定順序：離線 MemTest86 → 與 Valkyrie/Cthulhu 交換 CPU（或 Intel 保固 RMA）。freeze 時不會自動重開。

# 硬體 watchdog（2026-11-24 已啟用）

- Athena 沒有 ACPI WDAT；Ubuntu `/lib/modprobe.d/blacklist_linux_*.conf` 封鎖 `iTCO_wdt` 的自動載入，但明確載入可用（Z790 PCH TCO v6，上限約 613 s）。
- `/etc/modules-load.d/iTCO_wdt.conf` 開機載入模組；`/etc/systemd/system.conf.d/watchdog.conf` 設 `RuntimeWatchdogSec=5min`、`RebootWatchdogSec=10min`。用 `daemon-reload` 就生效，不需重開：`systemctl show -p WatchdogDevice` 為 `/dev/watchdog0`、`wdctl` timeout 300 s。
- 選 5 分鐘而非 30 秒：kdump 的 capture kernel 不會載入被封鎖的 iTCO，也不會餵它；太短會在寫 vmcore 時被 reset。
- 尚未驗證：重開後是否自動接管（下次開機看 `journalctl -b | grep "hardware watchdog"`），以及真的 freeze 時是否會 reset。回滾：刪掉這兩個檔案再 `daemon-reload`。

# 雜訊與時區修正（2026-11-24）

- Zeus 每分鐘 `:01` 由 `glances_collect.py` 以 `swear02` SSH 輪詢各機；共用 NFS `<admin-home>/.config/systemd/user/` 下的 `dvlab-windows-enroll.service` 與 `ninfer-oracle-tunnel.service` 原缺 `ConditionHost=zeus`（另殘留 `timers.target.wants/snap.firmware-updater.firmware-notifier.timer`），且 Athena 未開 `swear02` linger，導致每分鐘反覆啟停 `user@1075`（UID 1075 user manager）並刷 `fusermount3` AppArmor DENIED。
- 已在 Zeus 備份原 user units 至 `/var/backups/swear02-systemd-user-20261124/` 後為兩 unit 補上 `ConditionHost=zeus`、移除殘留 timer 連結並僅 `daemon-reload`（Zeus 四個運作中服務 `MainPID` 不變）；同步在 Valkyrie `daemon-reload` 與 `reset-failed` 清零失敗 unit；在 Athena 啟用 `loginctl enable-linger swear02` 並清理 user failed state，同時將 Athena 時區由 `Asia/Manila`（`PST`）改為 `Asia/Taipei`（`CST`）。驗證跨一輪 `:01`–`:15` 輪詢已無 `user@1075` 啟停、無 Zeus-only unit 失敗、無 `fusermount3` 拒絕紀錄，且 `overview.json`／`detail.json` 正常更新。
