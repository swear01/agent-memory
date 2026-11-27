---
title: VCS standard-cell 和 SRAM leaf 的唯讀 UCLI 觀測
scope: domains/eda
status: active
updated: 2026-11-28
---

- VCS Y-2026.03 的 gate-level post sim 雖有 `-debug_access+all`，`celldefine` SRAM leaf 仍可能回 `UCLI-CMD-NODBG`，file/line breakpoint 也無法設。先在 scratch 複製既有 executable 測 visibility。需要重新 elaboration 時，沿用同一 filelist、SDF、testbench 和 timing switches，增加 `-debug_region=cell+lib`，比對原本與新執行的 `sdfAnnotateInfo`。
- 被動追 notifier 分支可用 scoped file/line breakpoint 搭配 `stop -command` 與 `stop -continue`。用 `senv time` 記錄時間，`memory -write` 在相鄰原始碼行擷取前後全陣列，並以 `run -absolute` 限制到查驗窗。即使前後全 X，斷點仍能證明分支執行，資料損失則須看有無已知字轉 X。SCE-31 已用兩顆真實 SRAM leaf、未改 SDF 和 100 fs precision 驗證。
- VCS Y-2026.03 的 runtime `+no_notifier` 在整個設計停止 timing-check notifier register 更新。可沿用原執行檔做 A/B，SDF path delay 與 timing check／violation 訊息照常。保留 executable、SDF、TB 的 hash，另以 A/B 實際邊緣和 violation 訊息查對。這是全域資料效應診斷，範圍包含 standard cell 與 SRAM。
- 以 UCLI `dump -add <scope>.mem -aggregates` 保存 SRAM 陣列後，離線轉檔要用 `vpd2vcd +includemda trace.vpd trace-mda.vcd`，普通匯出會省略陣列。要保留同一 timestamp 的已知值→X，開 VPD `dump -deltaCycle on`，轉檔時保留同時間連續變化。先用 `+includemda -h` 核對陣列字數，再比較資料。
