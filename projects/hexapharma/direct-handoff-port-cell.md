---
title: "輸送 direct handoff 要核對接收 port 與進入方向"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 638d4072f78e0fa35624fab63c2a557906a8f1140f0b55079ac6aa7aef38d1dc
---

# 輸送 direct handoff 要核對接收 port 與進入方向

歷史助手把 outExit 等於下一台 inApproach 當 direct handoff，跳過中間必要 belt；使用者測量表中 default、mixed、dilute-chain 仍 deadlock，單台等其他案例成功。

只有輸出相鄰格就是接收 port 且方向符合才可省 belt，共享 approach 格仍需可接收的輸送帶。source 與 sink 邊界各自按其契約核對。後續描述加入 port／方向與調整模擬步數，但沒有新的全案例通過表，不能把早期測量的失败反轉為修復成功。
