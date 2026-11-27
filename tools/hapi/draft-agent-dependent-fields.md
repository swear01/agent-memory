---
title: "草稿 agent 正規化時同步重設相依欄位"
scope: "tools/hapi"
status: active
updated: 2026-09-09
evidence_digest: 057a58048bc3301be698080d2c7047d929c533411206527813042ccf5bc67596
---

# 草稿 agent 正規化時同步重設相依欄位

## Problem

歷史修正 commit 記錄：將過時的 agent 選項轉成可用值時，草稿相依欄位也需要重設。

## Durable rule

草稿的 agent 值被正規化或替換時，同步核對相依欄位，不只改主選項；用回歸測試涵蓋舊草稿載入後的一致性。

## Boundary

來源提供修正 commit 描述、typecheck 與 5 個 draft tests 通過的輸出；未提供完整 patch 或各相依欄位清單，因此不推定特定欄位或已部署。

歷史來源保留當時觀察，未重跑或驗證修復。
