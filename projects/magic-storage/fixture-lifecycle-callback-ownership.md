---
title: "Fixture 不應任意重複觸發 BlockEntity 生命週期"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: c112ed8dbe825cc52d9668077af7a61dc50013a52fd419929cb13d9f42b4216a
---

# Fixture 不應任意重複觸發 BlockEntity 生命週期

保存的 read-only review 指出 rollback fixture 在 Level#setBlockEntity 後手動呼叫 onLoad 並再次 rebuild network，與既有延遲驗證模式不同，可能重複 repository／network 活動。

按實際遊戲生命週期安排 fixture，等排程 callback 後再斷言，避免測試手動補呼叫掩蓋或製造問題。來源没有執行 Gradle，屬審查風險而非已重現故障；同份報告的 Oritech eligibility／catalog reload Major 是另一個未解機制，不能由此 fixture 建議宣稱整體通過。
