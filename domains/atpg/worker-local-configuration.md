---
title: "平行 worker 的前置設定要在實際物件上核對"
scope: domains/atpg
status: active
updated: 2026-09-16
evidence_digest: d90b2e0ebe638b60535afcbd0531062ae8eaefb008d2f812636e290223f0c78c
---

# 平行 worker 的前置設定要在實際物件上核對

歷史 progressive residual 輸出沒有 T2/T4 新增偵測；助手懷疑 master 的 nonscanDisconnectInfo 沒有傳到各 worker 自己的 Atpg 物件，並提出傳播修正。

追蹤 worker 建立、初始化與實際求解所用設定，不能假設 master 修改會自動套用到獨立物件。保留相同輸入的修正前後對照再判定因果；來源後续建置找不到 makefile，沒有修補重跑，不能說零增益已由此解釋。
