---
title: "擴充命令已註冊不代表晚期事件已送達"
scope: tools/pi
status: active
updated: 2026-09-16
evidence_digest: 5ba88cab4b4713cf06b90e262eabbb6f0d5ff3cf3c39adf0a15765bfa132a3d8
---

# 擴充命令已註冊不代表晚期事件已送達

歷史助手回報命令存在但 session name 未寫入；有載入套件的 session 沒有觀察到 agent_settled，較早未載入套件的 session 則有事件，因而懷疑套件阻塞。

分開核對套件載入、當前執行模式的事件發送、handler 進出與持久化結果。跨時間 session 的差異只能提供線索，不能證明套件或 LLM 呼叫造成阻塞；沒有 session_info 也不足以斷言 setter 從未被呼叫。來源沒有根因或修復重跑證據。
