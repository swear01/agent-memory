---
title: "自訂 transfer packet 須重查玩家當下存取權限"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: ef57482944ba838471f5fa17cb9484e263b3273ac246eefbcfcddf95ce51c56d
---

# 自訂 transfer packet 須重查玩家當下存取權限

同份 review 指出 held-container transfer 只驗 menu 和 slot 身份，漏掉 spectator 與 stillValid；開啟後切 spectator 或在 menu 尚未關閉前移遠，仍可能存取 core。

在真正執行 transfer 的 server 入口重查當下玩家與 menu 有效性；既有 container ID、state ID 與 slot 檢查不能替代這項權限。拒絕時 core 與游標都不得改變。

來源為靜態審查與測試設計，沒有修後實測；這與 listener 原子提交缺陷分開保存。
