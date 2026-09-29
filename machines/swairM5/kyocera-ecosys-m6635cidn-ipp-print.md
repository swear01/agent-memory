---
title: SwairM5 Kyocera ECOSYS M6635cidn 列印（IPP / 紙匣）
scope: machines/swairM5
status: active
created: 2026-09-29
updated: 2026-09-29
---

# Kyocera ECOSYS M6635cidn 列印注意事項

## 環境

- 機器：`SwairM5`（machineId `e02ab314-a679-4675-87b6-1d51914ed74f`）
- CUPS 佇列名：`Kyocera_ECOSYS_M6635cidn`（系統預設印表機）
- 驅動：AirPrint（`ECOSYS M6635cidn-AirPrint`）
- 位址：`192.168.1.100`（mDNS：`KMCC9F1B.local`）
- IPP：`ipp://192.168.1.100:631/ipp/print`（亦支援 `ipps://…:443/ipp/print`）

## Durable rule

1. **不要依賴 CUPS `InputSlot=`**：AirPrint PPD 的 `InputSlot tray-1/tray-2/by-pass-tray` 指令是空字串，`lp -o InputSlot=…` 送不到機器，常會落到 MP 匣錯誤。
2. **用紙匣時改走 IPP `Print-Job` + `media-col.media-source`**（`tray-1` / `tray-2` / `by-pass-tray` / `auto`），並帶齊 margin 與 `media-size`（A4 直向 `x=21000 y=29700`，單位 1/100 mm）。
3. **彩色**：`print-color-mode=color`；黑白：`monochrome`。
4. **紙匣 2（Cassette 2）** 是常用補紙匣；MP tray 空時若工作指定到 MP，印表機會進入 `stopped` +「Check for appropriate paper in MP tray」，需在面板取消/清除，或改送正確紙匣的新 IPP 工作。
5. **docx → PDF**：本機有 Microsoft Word 時，用 AppleScript `save as … file format format PDF` 匯出，再列印；輸出可放 `~/Downloads/`。

## 驗證過的例子（2026-09-29）

- 工讀生簽到退表（docx→PDF）與 `DV_Lab_115_座位表.pdf`（彩色）皆以 IPP `media-source=tray-2` 成功完成。
- 座位表 PDF Drive id：`1ByCnoHK8mH89L863D_qsau2FAf3P4uzV`；本機副本曾放 `~/Downloads/DV_Lab_115_座位表.pdf`。
