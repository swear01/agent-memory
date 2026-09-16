---
title: "排程平行 agent 前確認隔離方式可用"
scope: tools/agent-collaboration
status: active
updated: 2026-09-16
evidence_digest: eb30746f5c74e28bd28e2f2d3811dddb81eee2c49cf8118e3b7de3929e76ea7e
---

# 排程平行 agent 前確認隔離方式可用

Spawn 回覆不在 Git repo 且沒有 WorktreeCreate hooks，無法建立隔離工作樹；歷史助手於是改採循序整合。

先確認實際工作區與可用隔離方式，再安排會修改共享狀態的平行工作。條件不符時，可在保留現有成果與明確 ownership 下循序執行；不用為一個任務建立額外通用框架。來源沒有证明所有全庫 typecheck 都不能並行，也沒有整體後续完成結果。
