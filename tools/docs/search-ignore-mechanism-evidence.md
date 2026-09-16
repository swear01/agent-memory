---
title: "文件描述搜尋排除機制前先核對真實設定"
scope: tools/docs
status: active
updated: 2026-09-16
evidence_digest: 64e0ffdeb4e22d6e943befec3d1a302d8452e8a62d90d8e24219e0d5e83cd8f4
---

# 文件描述搜尋排除機制前先核對真實設定

歷史文件稱 rg 依不存在的 .rgignore 排除 archive；助手複查後說實際來自 .gitignore，並撤回把刻意保留的 PLAN 與 archive 系統說明判為違規的早期 finding。

分開實際搜尋行為與提供該行為的設定，按本 repo 規則及明文豁免審查。來源只列修正方向，沒有檔案修改或新驗證；也不能從歷史說明推定目前 ignore 檔仍相同，或把所有 archive 字樣視為 active 引用。
