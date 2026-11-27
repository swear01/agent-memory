---
title: "CVSD SCE 串流編碼的 writer FIFO 容量下界"
scope: "projects/cvsd-final-sce"
status: active
updated: 2026-11-28
---

- 尾列的 `j + ancestor` 嚴格遞增，可以提前封口 bitmap bytes。每 bank 的 record 地址順序仍可能要求尚未產生的 directory 先 append。量容量時，先用無限 SRAM 頻寬及無限 FIFO 算因果下界，將已完成卻無法 append 的 bytes 計入 staging。接著加回原 ring credit 與實體寫拍，逐拍計 staging、未封口 byte、超過寫入寬度的 spill。零拍差需要同時核對 advancing 與 freeze edges，以及同 bank 的原寫入期限。
- SCE-20 全掃的自然反例已超過 16 B，即使准許 SRAM 提前寫。提前填入原 ring 尚未 append 的地址會改變 writer 狀態及仲裁，應另算 valid、地址與補寫成本。固定原寫拍的 replay 只能證明該組交付時刻；它不會自動驗證 occupancy 驅動的 hold 控制。重播與結果在 SCE repo 的 `exp/area/as07-stream-encoder/report.md`。
