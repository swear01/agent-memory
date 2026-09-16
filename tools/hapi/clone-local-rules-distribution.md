---
title: "不改 upstream 的規則要有明確本地載入入口"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: d3f78879d95ea11a1ec1963fb6492233c2aff6ed7b7e9dee91883e39e87ab9bf
---

# 不改 upstream 的規則要有明確本地載入入口

使用者要求不改 upstream 規則、workflow 與 review prompt；歷史方案指出 common git dir 的 info 本地筆記可供 linked worktrees 共用，但相鄰早期方案仍提議改 upstream。

以當次範圍為準，明確讓相關 skill 載入本地筆記；換 clone／機器仍需私人同步，不能把 Git 未追蹤的規則當已跨機發布。文件指示也不是 GitHub required check，來源只提出配置方向，沒有新的 ruleset 或 workflow 啟用證據。
