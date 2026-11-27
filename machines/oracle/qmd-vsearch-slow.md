---
title: oracle 上 qmd vsearch 慢的原因是 1.7B query expansion，不是向量比對
scope: machines/oracle/qmd-vsearch-slow
project: agent-memory
status: active
confidence: high
evidence: 2026-10-22 在 oracle（Neoverse-N1 2 vCPU、11 GB、qmd 2.8.3、CPU-only llama.cpp）上分階段計時，並讀過 dist/store.js 的 vectorSearchQuery 和 dist/llm.js 的 expandQuery。
created: 2026-10-22
updated: 2026-11-27
tags:
  - qmd
  - oracle
  - performance
---

# Oracle 的 QMD 卡在 CPU query expansion

- Oracle 用 `shared-memory` skill 的 config/cache/model 前綴；漏前綴出現 `No collections` 時不要往預設 index 新增 collection。Vsearch 在 QMD 2.8.3 先跑 1.7B expansion，再對原始和 vec/hyde 變體依序 embed，最後才比對 sqlite-vec。SwairM5 的 2.9.0 本機 `vectorSearchQuery` 還有這個 expansion。CLI 名稱和向量比對耗時不能解釋整個 query latency。Expansion 沒成功就不會留下 cache，下次 timeout 重試仍得重算。
- 明確關鍵字用 `qmd search -c memory`，在 Oracle 是秒級，無需 embedding。需要 pure authored vector 可用 `qmd query 'vec: <查詢>' --no-rerank` 避開 expansion；短中文問題仍可能排序差。Oracle 冷啟 embed 有成本，完整 hybrid 用有 GPU 的 Mac。Oracle 例行只 update；大批 embed 給 GPU 主機。
- Mac 和 Oracle 用同一 embeddinggemma-300M URI 和 bytes，實際 `models.embed` 設定可覆蓋環境變數。來源用只含應分享 collections 的 quiescent index，不混入額外本機 collections；目的端以 repo 內容重新 update，驗 memory 文件、vectors 和 pending。全索引 pending 可包含其他 collections，不等於 memory 全部待補。見 [通用 snapshot 規則](../../domains/llm-inference/qmd-snapshot-portability.md)。
- 無法 Mac SSH 時，曾用 Tailscale file 送已停寫的 SQLite；接收端 access denied 需核對 Tailscale operator，再 `tailscale file get`。安裝前確認無 writer、保存舊 index set，來源主檔與 WAL/SHM 不可混配，裝入後依本機路徑驗一次 update 和 retrieval。舊快照的副本不與活 DB 共用，不在 Oracle 重跑全體向量作為修復。

[整理前的測量與逐次驗證紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/oracle/qmd-vsearch-slow.md)
