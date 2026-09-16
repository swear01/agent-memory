---
title: "Atomic replace 不能保護共用的固定暫存檔"
scope: tools/python
status: active
updated: 2026-09-16
evidence_digest: 1c705dd9768ed373652bb4d42df193997f85a790cf88d01fed214f414e017a79
---

# Atomic replace 不能保護共用的固定暫存檔

來源有 detail.json.tmp 的 os.replace FileNotFoundError 與 JSONDecodeError；歷史助手將它歸為 cron 和手動執行共用暫存檔的競爭，並提出改用含 PID 的名稱。

同時執行者應各自安全建立目的目錄內的暫存檔，再原子替換；固定名稱即使最後 rename 原子化，前面的寫入仍可能互相干擾。不能把解析錯誤稱為無害，也不能假定 cron 永不重疊或 PID 名稱就絕對唯一。來源僅提出修正，未證實競爭消失。
