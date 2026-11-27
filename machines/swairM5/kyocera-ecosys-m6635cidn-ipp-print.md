---
title: SwairM5 Kyocera ECOSYS M6635cidn 列印（IPP / 紙匣）
scope: machines/swairM5
status: active
created: 2026-09-29
updated: 2026-10-20
---

# Kyocera ECOSYS M6635cidn 列印

- **每份先核對再送印。** 列出檔名、頁數、單／雙面、黑白／彩色等設定，等使用者明確同意該份。後來找到的教材、考古題或自製附件不算原印件。
- SwairM5 CUPS 佇列 `Kyocera_ECOSYS_M6635cidn` 使用 AirPrint，印表機 `ipp://192.168.1.100:631/ipp/print`（也支援 IPPS）。先查本機是否直接可達。需 VPN 時用原生 [DVLab IKEv2](dvlab-ikev2-shortcuts.md)，以 `route -n get 192.168.1.100` 查預期介面；`scutil --nc` 未列 VPN、捷徑成功或 mDNS 無結果都不足以判定網路。用 `Get-Printer-Attributes` 確認型號和 UUID。
- AirPrint PPD 的 `InputSlot` 可為空指令，`lp -o InputSlot=…` 不保證設備收到指定紙匣。需要精確紙匣時，用 IPP `Validate-Job` 再 `Print-Job`，指定 `media-col.media-source=tray-2` 或實際裝紙匣，以及 A4 `media-size`（21000×29700，單位 1/100 mm）。MP tray 錯誤會令裝置 stopped；先清除卡住工作並確定是否需要另送，避免雙印。
- 明確指定 `print-color-mode=monochrome` 或 `color`；雙面長邊用 `sides=two-sided-long-edge`，原尺寸用 `print-scaling=none`。CUPS 可另用 `Duplex=DuplexNoTumble`／`ColorModel=Gray`／`PageSize=A4`，仍不依賴 PPD 的 InputSlot。DOCX 需先用 Word 匯出 PDF 並查實際頁數。
- 用印表機 `Get-Job-Attributes` 查 `job-state=completed`、`job-completed-successfully` 與 `job-impressions-completed` 是否等於文件頁數乘份數。CUPS accepted／completed 與 HTTP 成功都不證明出紙；sandbox 無法查 IPP 時回報證據限制，不重送來測。和舊卡件分開核對本次 job ID。

[逐次列印、設備證據與當時限制](https://github.com/swear01/agent-memory/blob/93786b66aa982bda81d4c1ebe6cfceaa832b5d32/machines/swairM5/kyocera-ecosys-m6635cidn-ipp-print.md)
