---
title: "Solver 選到特殊操作不代表題目強制需要它"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 0d5bf1e424159393f6b8e871b3a3e2125f47fd526e7f15934bfdd80807d487f9
---

# Solver 選到特殊操作不代表題目強制需要它

歷史探索先把 BFS 選用 skew 視為跨圖解耦成功，隨後發現同長 forward-only 也能治癒；加重 hazard 的兩種布局又讓完整與 forward-only solver 都無解。

分開驗證完整解存在與受限操作真的無解，不能由 expansion order 或 difficulty bonus 推定必要機制。相同起點／幾何與一致位移下的對称性是該 catalog 的分析，壁面可造成掃描分歧但單一 wall 仍不足以強制特定操作。來源尚在重新設計，沒有最後成功 gadget，也不外推所有模型都只能靠牆解耦。
