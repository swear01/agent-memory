---
title: 用大量 Haiku subagent 解題來驗證參考文件（大抄、速查表）
scope: global
tags: [claude-code, subagent, haiku, verification, cheat-sheet]
status: active
created: 2026-11-03
updated: 2026-11-03
---

使用者確認有效且省錢的做法（LSV 期中大抄，2026-11 共跑約 130 個 Haiku 解題 + 7 個逐區審查）。

## 做法

- 建一個精簡 custom agent（`~/.claude/agents/haiku-solver.md`）：`model: haiku`、`tools: Read, Bash`、`disallowedTools: mcp__*`、`maxTurns: 15`；system prompt 只說「少用工具、python 只印結果、不改檔、照指定格式回報」。
- **一個 agent 只解一題真題**（歷年作業原題，而不是自己出的題：自己出的題曾出現 onset/offset 重疊的錯題）。prompt 指定題目檔與題號、**只准讀**文件的哪幾個分區，回報 ≤200 字：`ANSWER / LINES USED (file:line) / WRONG (python-checked) / MISSING / UNCLEAR`。
- 另開「逐區審查」agent：讀一個分區 + 相關題目，回報 `REDUNDANT / MISSING / WRONG`，用來找可刪的冗餘。
- 同時上限 20 個 subagent（`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`），超過就有空位再補。
- 讀結果用 agent 的 hand-back 訊息；不要去讀 `tasks/*.output`（完整 JSONL transcript，會塞爆 context）。

## 必守的驗證規則

- Haiku 的 WRONG 常是誤讀（例如定義不同、題目 PDF 轉文字時欄位跑掉、把 don't care 當 onset）。**每個報錯都要主 agent 自己用 python 暴力驗證或對官方解答**再改。
- 實際抓到的真錯多半是：抄錯官方範例（子句編號、步驟順序）、規則漏前提（退化情況）、邊界條件（如 n=1）。
- MISSING 先檢查是否其實在別的分區；審查 agent 只讀一區，常把別區已有的內容報成缺漏。
- 每輪改完重新排版並檢查（頁數、溢出、邊界）。

## 為何有效

Haiku 便宜，可大量平行；「只能用文件解題」能直接暴露文件的缺口與錯誤，比單純請模型審稿更具體。主 agent 只做驗證與修改，成本集中在判斷上。
