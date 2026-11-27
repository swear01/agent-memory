---
title: "容器資料槽契約必須符合實際封包寬度"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 7b18390bae8c393c3981751c3a491994b20ad7da37d5ff6c511e5cae45624da9
---

# 容器資料槽契約必須符合實際封包寬度

Minecraft 1.21.1 容器資料封包使用 `writeShort/readShort`。設計 data slot 時核實際序列化寬度與 signedness，讓邊界值走完整 wire codec，直接呼叫接收函式不足以驗證。64-bit 拆分與 enum 順序仍需源碼及實測支持。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/magic-storage/data-slot-wire-width.md)
