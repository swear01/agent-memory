---
title: "Build 通過與舊版可啟動都不能證明新版已修好"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 2a888d003f82a9bfd1ca643c2d70f6a88752c824eba064a05230a2f4165301b6
---

# Build 通過與舊版可啟動都不能證明新版已修好

歷史助手先推測 Mixin 介面非 public 是啟動根因；後續新版仍未到 READY，回報同 instance 的舊版可啟動，接著部署修正版準備驗證。

用相同條件比較版本並縮小變更，再以正式 runner 的實際啟動結果驗收。關閉全螢幕仍失敗只限制該假設，不能順帶證實或否定介面權限根因；focused test、build 與產物 hash 一致也不能代替 READY。來源沒有新版最終成功啟動證據。
