---
title: VCS standard-cell 和 SRAM leaf 的唯讀 UCLI 觀測
scope: domains/eda
status: active
updated: 2026-11-28
---

- VCS Y-2026.03 的 gate-level post sim 雖有 `-debug_access+all`，`celldefine` SRAM leaf 仍可能回 `UCLI-CMD-NODBG`，file/line breakpoint 也無法設。先在 scratch 複製既有 executable 測 visibility。需要重新 elaboration 時，沿用同一 filelist、SDF、testbench 和 timing switches，增加 `-debug_region=cell+lib`，比對原本與新執行的 `sdfAnnotateInfo`。
- 被動追 notifier 分支可用 scoped file/line breakpoint 搭配 `stop -command` 與 `stop -continue`。用 `senv time` 記錄時間，`memory -write` 在相鄰原始碼行擷取前後全陣列，並以 `run -absolute` 限制到查驗窗。即使前後全 X，斷點仍能證明分支執行，資料損失則須看有無已知字轉 X。SCE-31 已用兩顆真實 SRAM leaf、未改 SDF 和 100 fs precision 驗證。
