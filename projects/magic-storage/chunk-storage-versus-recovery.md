---
title: "破壞時 recovery 快照不解決常態 chunk 庫存大小"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: e55dccffafb75fc8a269428d274dffbda65ae2545b7adfd96b08f14ed369eac2
---

# 破壞時 recovery 快照不解決常態 chunk 庫存大小

使用者問庫存是否仍存 chunk；歷史助手先說明掉落 Core 的一次性 recovery token，後來才承認常態庫存仍由 BlockEntity saveAdditional 写入 chunk NBT，破壞時 SavedData 只另存保險快照。

分開追查常態保存、掉落攜帶內容及故障恢復三條資料路徑，不能用掉落物只有 token 回答 chunk 是否膨脹。移出 chunk 需另外設計世界格式與遷移；來源沒有 AE2 對照或遷移完成證據。claim 早於還原例外的風險也仍保留，不能稱 token 自動保證原子恢復。
