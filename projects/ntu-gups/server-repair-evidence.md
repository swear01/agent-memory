---
title: NTU GUPS workstation repair and identity verification
scope: projects/ntu-gups
status: repair_requested_diagnosis_pending
updated: 2026-10-14
---

- **Dock 有線連線後 SSH 可用，卻可能連錯工作站。** 先以 `system_profiler SPUSBHostDataType` 對照 USB 網卡與 BSD 介面，再核 carrier、鄰居、hostname、網卡和已知 SSH host key。舊 `SPUSBDataType` 空輸出不能證沒辨識。無 DHCP 位址時可先在同介面做 IPv6 link-local 鄰居探索，保留 scope；這次其實連到原已知主機的其他網孔。供電燈亮、顯示與網路不回應都不足以確認成功開機或確診零件。
- **Apple Mail 草稿應有兩張照片，讀出 MIME 只見一張。** 待附件載入完成，再讀保存的 MIME，對照核准正文、收件及回覆信串、全部附件的原始 bytes。Rich text 的 U+2028、純文字的引文符號及附件占位均不代表實際 HTML 正文改變；仍要核正本。寄件備份只證明本機寄送的內容，供應商收件與維修另查。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/ntu-gups/server-repair-evidence.md)
