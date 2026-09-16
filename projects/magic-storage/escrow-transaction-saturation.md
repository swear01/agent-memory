---
title: "Escrow 回填要涵蓋飽和時的 commit 與 rollback"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: cb56233226ff4959e9d454f9bfd6ea49639f2220eb41067a93ae400a0ef69e2f
---

# Escrow 回填要涵蓋飽和時的 commit 與 rollback

保留的 subagent 審查報告指出，瓶數達 Long.MAX_VALUE 時抽取後立即回填，可能讓消耗又產出瓶子的合成交易預檢通過、實際插入與回滾卻失敗；助手僅接續提出交易層測試與修正。

資源 escrow 的測試要涵蓋最外層交易、容量飽和、提交及回滾，不能用直接抽取成功代替。回填時機須由實際交易契約決定。這是歷史 review finding，沒有執行重現或修復後通過證據，不宣稱已確認永久物品損失。
