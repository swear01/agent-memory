---
title: Issue 25 preprocessing fail-closed boundaries
scope: project
project: issue25-memory-promotion
tags: [redaction, qmd, tokenizer, sqlite, security]
status: active
created: 2026-08-30
updated: 2026-08-30
---

- **普通 redaction scan 綠，但 episode 包含 image/audio/video data URI。** 拒絕殘留 `data:` 媒體、runtime IDs、原始圖片 payload 與空 media containers。v2 去 bytes 仍留下 `input_image` metadata，故 v1/v2 shard 都不得 assembly 或 embed。v4 覆蓋 OpenAI image_url/b64_json/input_image、Anthropic image source、Gemini inline MIME/data 和嵌套 JSON，同時保留 captions 與 input-text siblings。原始私人 harvest 不重寫。
- **Unicode splitter 最後才檢查大小，tokenizer 先吃多 MB。** 以字元上限先縮窄 binary-search candidate，再建立或 tokenize 字串；v4 上限 2,700 字元。驗 multi-MB 重複 fixture 近線性、精確 code-point provenance、每次 tokenizer input 不超 cap；舊 SIGSEGV 無 core，不能定成精確 llama.cpp 指令。
- **SSH 斷線或重開後不知道應從哪續跑。** 長跑由 enabled user service 所有，以 exact contract、bounded restart／人工重試及已完成 ExecCondition 管理。source DB 逐 source commit；真正需要更細 resume 時，用 private per-source SQLite 暫存 ordered chunk index、immutable source/contract hash、next record/code-point/overlap 游標、text hash 和 UInt32LE token BLOB。續跑先完整驗 staged prefix，再原子匯入同 digest 的 complete source，驗中斷／不中斷相同。
- **SQLite quick_check／重建通過就發布索引。** 跨 shard 依 canonical source order dedup exact inputs，驗 token bytes，保留第一 representative。完整 reconstruction、source/hash、FTS/joins、token/provenance、殘留媒體與整體安全、backend parity、retrieval 是不同 gates。進度 reference 用 `SUM(issue25_sources.reference_count)`，勿把 contextual `issue25_occurrences` 當分母。舊服務與 timings 看已完成的私有報告；退役前依 [替代檢索與引用查核](retired-artifact-closeout.md)。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/issue25-memory-promotion/preprocessing-fail-closed-boundaries.md)
