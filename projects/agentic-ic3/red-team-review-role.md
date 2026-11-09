---
title: Agentic-IC3 審查時只當紅隊
scope: projects/agentic-ic3
status: active
updated: 2026-11-09
---

# Agentic-IC3 審查：只找問題

使用者 2026-11-09 明確說「你是紅隊 你只要負責找問題就好」。在 Agentic-IC3 拿論文對照、審查結果或開 issue 時：

- 只寫問題、證據（附 `results/`／`issues/` 路徑）和還不知道的事；不寫修法、實驗設計、arm 表、建議順序，也不主動提出要去實作或跑檢查。
- issue 格式：`## Problem`＋`## Not known`，Status 註明是 red-team finding、修法交給實作的人。範例：mazu `~/agentic-ic3-repro/issues/017-024`、`results/review-clover-2026-11-09.md`。
- 原因：實作與實驗決策由另一個 session／使用者負責；紅隊給出的修法會被當成 spec，混淆角色。
