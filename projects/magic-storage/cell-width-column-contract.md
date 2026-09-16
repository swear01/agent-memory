---
title: "最小 cell 尺寸變動須更新欄數契約"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 5cfc4a7ce8bb6e701f023b636e5cd521b6d0c2adc6c4b3067de599a145833e69
---

# 最小 cell 尺寸變動須更新欄數契約

歷史助手回報 Base GameTest 通過，但 SelfTest 在最小 cell 改為 56px 後仍要求窄視窗四欄，造成尺寸組合失敗。當時指出正確方向是窄時減少欄數，同時保留後續項目。

尺寸參數變動時，同步檢查寬度到欄數的推導、全部項目的可達性及測試邊界。不能只因另一套 runtime 測試通過就忽略佈局失敗，也不能把目前錯誤座標直接當新預期。來源停在診斷與修正方向，沒有修正後重跑結果。
