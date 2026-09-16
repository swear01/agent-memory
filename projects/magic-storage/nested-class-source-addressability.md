---
title: "具名 nested class 仍須核對整條 owner chain"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 2d3ee125f6c8b78ffbb0616cc0f16b7f4308c89ad651e71207ce18a59f196558
---

# 具名 nested class 仍須核對整條 owner chain

保存的 subagent 報告指出 anonymous owner 下的具名 Key class 被 classifier 當成無法解析的 source name；修正沿 InnerClasses owner chain 排除 anonymous／local／synthetic owner 下的候選，並把 classifier cache 由 2 升為 3。

Source-addressability 不能只看末端類名，語意分類改變時也須使旧快取失效。來源回報指定 RED→GREEN 與 211 項完整測試，首次缺 ancestry artifact 先補 prerequisite 再重跑；沒有實際 ATM10／IE 掃描、migration 或 PR 發布證據。
