---
title: "儲存查詢先確認資料由 Hub 還是 Runner 持有"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 30964f3fa293f98d108344cf6ee317a7f5b6eedb53e28c81c05d1a9d744ccdf2
---

# 儲存查詢先確認資料由 Hub 還是 Runner 持有

歷史規劃曾把每個 Runner 當成 HAPI SQLite 的擁有者；助手後來更正 hapi.db 由 Hub 使用，Runner 保存自己的狀態、設定與 logs，其啟動的 agent 資料庫是另一種資源。

先確認資料的實際擁有元件，再設計查詢路徑。若只查 HAPI DB 空間，應由 Hub 查其已設定的資料庫及相關 WAL/SHM，不為不存在的 per-runner HAPI DB 增加 RPC。來源只到修正 issue 與設計，明言未實作；不代表儲存頁已部署。
