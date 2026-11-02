---
title: DeepSeek API 尖峰/離峰計價時段（2026-08 起）
scope: tools/llm
tool: deepseek
status: active
confidence: high
created: 2026-11-02
updated: 2026-11-02
tags:
  - deepseek
  - pricing
  - off-peak
  - 離峰
  - scheduling
---

# 規則

來源：DeepSeek 官方價目頁 `https://api-docs.deepseek.com/quick_start/pricing`（2026-11-02 查證）。
2026-08-16 16:00 UTC 起生效。

- **尖峰**：週一到週五 01:00–04:00 與 06:00–10:00 UTC，
  也就是北京／台灣時間（UTC+8）**09:00–12:00 與 14:00–18:00**。每天共 7 小時。
- **離峰**：其餘所有時間，包含週末全天與中國法定假日全天。
- 離峰價是尖峰價的一半（deepseek-flash、deepseek-v4-pro 都適用）。

使用者說「DeepSeek 要在離峰時間跑」時，指的就是這個費用離峰：只要避開平日上述 7 小時即可，
晚上、清晨、週末都能跑。排程時一個 run 要能在下一個尖峰開始前跑完才開。

# 不要用的舊時段

`16:30–00:30 UTC`（UTC+8 的 **00:30–08:30**）是舊 V3/R1 的夜間折扣，已停用。
2026-11-02 曾憑模型記憶直接用了這個舊時段，沒查官方頁，結果讓 agentic-ic3 的 gate
在真正的離峰（平日晚上）乾等，被使用者糾正。

**教訓**：價格、時段、配額這類會變的商業規則，一律先查官方頁再用，不要憑記憶；
寫進排程或轉交給其他 session 前也要附上來源。
