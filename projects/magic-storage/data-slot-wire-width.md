---
title: "容器資料槽契約必須符合實際封包寬度"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 7b18390bae8c393c3981751c3a491994b20ad7da37d5ff6c511e5cae45624da9
---

# 容器資料槽契約必須符合實際封包寬度

保留的架構審查片段指出 Minecraft 1.21.1 容器資料封包使用 writeShort/readShort，並警告當時提出的 data-slot 契約不安全。片段在後續說明前即截斷。

設計同步資料時，以實際序列化寬度與 signedness 核對編碼和解碼；直接呼叫資料接收函式的測試不能代替走過 wire codec 的驗證。此來源不足以支持草稿加入的完整64-bit拆分方案、enum順序或修補成功結論，這些仍保留待查。
