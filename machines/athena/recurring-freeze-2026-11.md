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
- 現在的死法是整機 freeze，不是 panic：`oops=panic panic=10` 與 kdump（`kexec_crash_loaded=1`）都沒觸發，pstore、`/var/crash` 皆空。11-23 23:55 死後停了 13 小時才被人開機；11-24 14:22 死、14:33 回來。
- 11-24 的兩次開機都有 `BERT: [Hardware Error]: Skipped 1 error records`，代表韌體記下前一次 boot 的硬體錯誤（記錄 >1 KiB 所以 kernel 沒印內容）；更早的 boot 沒有 BERT 記錄。
- 對照：Valkyrie、Cthulhu（同 i9-13900K、microcode `0x133`、kernel `7.0.0-43`）與 Mazu（14900K）自 10-01 起 `softirq: huh` 與 BERT 都是 0。證據指向 Athena 單機 CPU／RAM／主機板，Raptor Lake 已發生的 Vmin 退化最可疑（microcode 只能防止惡化，不會修復），但尚未經硬體交換確診。
- 雜訊非根因：fusermount3 AppArmor DENIED 每 10 秒刷一次；earlyoom 顯示記憶體 99% 空閒，無 OOM、Xid、thermal。
- 下一步沿用既定順序：離線 MemTest86 → 與 Valkyrie/Cthulhu 交換 CPU（或 Intel 保固 RMA）。freeze 時不會自動重開，可考慮 iTCO 硬體 watchdog + systemd `RuntimeWatchdogSec` 讓它自己 reset。
