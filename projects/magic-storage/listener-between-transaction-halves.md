---
title: "Listener 不能插在資源與容器兩段提交之間"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: ef57482944ba838471f5fa17cb9484e263b3273ac246eefbcfcddf95ce51c56d
---

# Listener 不能插在資源與容器兩段提交之間

歷史唯讀 review 指出 core ledger 修改後同步呼叫 listener，返回才套用 held container；listener 拋錯可使水已扣除但仍拿空桶，反向則可能複製資源，重入也可能使預算 placement 過時。

把 core 與容器當同一提交邊界設計，明確處理失敗及重入；測試中途例外後只能保持原狀或完成整筆轉移。不能靠聲稱 listener 不會失敗來建立原子性。

來源給出可重現步驟與建議 RED，但明示未建立那些測試、未重跑 GameTest；不把既有 compile 和單元綠燈當這條路徑已修復。
