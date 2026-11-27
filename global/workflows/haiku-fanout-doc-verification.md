---
title: 讓 Haiku 用真題驗證大抄，逐條查證模型報錯
scope: global/workflows
tags: [claude-code, subagent, haiku, verification, cheat-sheet]
status: active
created: 2026-11-03
updated: 2026-11-28
---

- 審查大抄時，每個 solver 只拿一道歷年真題和指定的文件區段。要求短答 `ANSWER / LINES USED (file:line) / WRONG / MISSING / UNCLEAR`，限制工具為讀檔及必要的本機計算，不改稿。另按頁段找同義重複、漏前提與錯例，使用有界並行並讀 hand-back 摘要。
- 模型報 `WRONG` 時先用 Python 獨立算或對官方解答。曾因 PDF 欄位錯位、定義不同、把 don't care 算成 onset 而誤報。模型只讀一區時，`MISSING` 要先查完整文件。
- 修完再驗排版、頁數、邊界與溢出；區分對照真題跑到的條件和未涵蓋的變形。[原始分派與反例](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/global/workflows/haiku-fanout-doc-verification.md)
