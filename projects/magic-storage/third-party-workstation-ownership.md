---
title: "Storage 模組不代管第三方工作站註冊"
scope: projects/magic-storage
status: active
updated: 2026-09-08
evidence_digest: 17eaedadabd02015384a2587a31aa9cceae4170e64966df2eea9bf009122d27a
---

# Storage 模組不代管第三方工作站註冊

## Problem

使用者拒絕讓 storage 模組把第三方 Furnace descriptor variants 註冊到 EMI Smelting workstation，要求先查明附屬模組與整合責任。

## Durable rule

相容性缺項先調查擁有該 recipe／workstation 的模組及其整合，依公開介面確認責任。不要在 storage 模組加入原本屬於第三方的註冊來掩蓋缺項。

## Boundary

這是 Magic Storage 的模組責任邊界。來源含明確使用者糾正，但未證明缺項一定來自少裝附屬模組或 tag，也未證明修復完成。

## Verification

控制者核對保留的歷史訊息及同來源鄰近訊息，確認上述使用者糾正。未重新執行歷史測試，不把助手陳述或修復計畫當成已驗證的結果。
