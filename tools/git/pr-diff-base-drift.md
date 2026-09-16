---
title: "PR-only patch 要排除基底漂移"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 8fd0c239b7f445a1a80e8bdce351d163e075a1e12e1800ec980591a622ca79a3
---

# PR-only patch 要排除基底漂移

歷史助手發現用舊固定 base 到 PR head 的 diff 帶入 drift，會還原新 commit，於是改以 merge-base 產生 PR-only 差異。

先確認真正分歧點、目標基底與預期 commit 集合，再檢查產生的 patch 不包含無關回退。Merge-base 是比較起點，不能保證整個分支沒有其他改動。來源另記 reset 後遺失產物與複製舊解析，這些獨立風險仍留在原始帳本；沒有最終套用成功證據。
