---
title: Linux 將中間邏輯 CPU 下線會導致 HAPI 內建 Bun os.cpus() 崩潰重啟
scope: tools/hapi
status: active
updated: 2026-11-24
tags: [hapi, bun, linux, cpu, runner]
---

# 症狀與根因

- 在 Linux HAPI runner 主機將非末端邏輯核心（例如 `/sys/devices/system/cpu/cpu1/online` 設為 `0`，使在線清單變成 `0,2-31`）熱拔除後，`hapi-runner` 會在連線後執行 `collectMachineHealth` → `sumCpuTimes` → `os.cpus()[0].times` 時立刻拋出未捕捉例外並退出，導致 user systemd 陷入每 5 秒 `activating (auto-restart)` 迴圈：
  `FATAL: Uncaught exception {"code":"ERR_SYSTEM_ERROR","errno":0}` / `Error: Failed to get CPU information at populate (node:os:18:25)`。
- 以 `BUN_BE_BUN=1 hapi -e 'require("node:os").cpus()[0].times'` 對照驗證（Bun `v1.4.4` Linux x64）：單純用 `taskset -c 2-31` 或 systemd `AllowedCPUs=2-31` 限制親和性時，`os.cpus()[0].times` 完全正常；只有當中間核心 `online=0` 造成 sysfs CPU 編號不連續時才會失敗。

# 處置原則

- 在需要隔離故障核心但仍要執行 HAPI runner 的 Linux 主機上，維持所有邏輯核心 `online=1`，改用 systemd `CPUAffinity=`、`AllowedCPUs=`（`init.scope`、`system.slice`、`user.slice`、`machine.slice`）、IRQ `smp_affinity` 導流，以及將目標核心 `cpufreq/scaling_max_freq` 鎖定在無 Turbo 基頻（例如見 [Athena P-Core 0 隔離紀錄](../../machines/athena/recurring-freeze-2026-11.md)）。
