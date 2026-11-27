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

# Mazu kernel panic：SCHED_SOFTIRQ、pstore 與遠端恢復界線

- 查 previous-boot journal 與 pstore。9/5 的 `BUG: scheduling while atomic` 在 `swapper/8`，隨後 timer IRQ/SCHED_SOFTIRQ 發生 NX stack execution；同 CPU、時間戳和 trace 的 Oops/Panic 記錄是同一事故階段。 `always_kmsg_dump=N`，此記錄不是正常關機流程。可採 `sched_balance_update_blocked_averages → sched_balance_softirq → timer IRQ` 外層 stack，損壞的內層 frames 不能單獨定根因。
- LAN、SSH、Hub 或公開入口同時不通，不能單由症狀判成已關機。Mazu 當時 panic／freeze 的 WoL 重試沒有恢復；WoL 不等於硬體 Reset，也無法證明正常關機後不能喚醒。必要時由有權限的現場／硬體管理入口處理。復原後驗 Runner、Hub、Tunnel 與本機及公開 HTTP。重新開機政策查實際 `kernel.panic`／`panic_on_oops` 和 watchdog，而非沿用 9/5 快照。
- 重開後高負載的 package 高溫／throttle 要獨立處理，不能回推當時是 thermal shutdown。當時 pre-panic 多為 idle，沒有可見 OOM、Xid 或 thermal critical；NVIDIA taint 只是曾載 module，未在 call trace。無 MCE 也不能排除 CPU/RAM/主機板 silent corruption。
- 與 [Athena 後續 freeze](../athena/recurring-freeze-2026-11.md) 對照有相似 SCHED_SOFTIRQ 指紋，單次 trace 不能斷言同根因。若要定位缺失的證據，先確認 kdump 與 `kexec_crash_loaded=1`，Mazu 當時沒有 vmcore。應在核准維護窗口用已知可回復的 kernel 做 A/B，再做離線記憶體和硬體隔離；有新版核心不代表該缺陷已修。

[9/5 原始時間線、pstore 與熱節流對照](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/mazu/kernel-panic-2026-09-05.md)
