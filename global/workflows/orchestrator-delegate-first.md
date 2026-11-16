---
title: Orchestrator session 先把任務交給子 session
scope: global/workflows
status: active
updated: 2026-11-17
tags: [orchestrator, delegate, start_session, subagent, sonnet]
---

# Orchestrator session 先委派

2026-11-17，使用者明確要求記住：當 session 擔任 orchestrator 時，使用者交辦的任何任務，第一步先交給子 session（Claude Desktop `start_session`，`model: "claude-sonnet-5-5"`、`context: "fresh"`）。主 session 只負責協調、整合與回報。

例外：單一工具呼叫就能解決，或已接近完成的事，直接在原 session 做。

相關：一般 subagent 分工見 [強制 subagent 用 Sonnet](../../tools/claude-code/subagent-model-force.md)。
