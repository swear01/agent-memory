---
title: QMD vectors can be distributed as quiescent local snapshots
scope: domains/llm-inference
tool: qmd
status: active
confidence: high
evidence: QMD 2.8.3 source review plus a GPU-to-CPU snapshot pilot.
created: 2026-08-18
updated: 2026-08-19
tags:
  - qmd
  - embeddings
  - sqlite
  - gpu
  - cpu
  - fleet
source_refs:
  - public-release:qmd-snapshot-portability
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# What can be shared

The Qwen3-Embedding-0.6B Q8_0 GGUF model can be copied between compatible machines when the file checksum matches. Copy the model file, not native `node_modules` or platform-specific runtime binaries.

A completed QMD SQLite index can also be copied as a **quiescent local snapshot**. QMD vectors are stored in SQLite/sqlite-vec and do not contain GPU, CUDA, Vulkan, Metal, or VRAM state. A CPU machine can search vectors produced on a GPU machine.

# Snapshot requirements

Only snapshot after all QMD processes have exited. If `index.sqlite-wal` or `index.sqlite-shm` exists, treat the main file and sidecars as one coherent snapshot; never copy only the main file from a live WAL database. Do not put one live QMD SQLite database on shared storage or sync multiple writers into it.

The destination needs the same QMD/sqlite-vec compatibility, the exact embedding model URI and model bytes, and a matching Markdown corpus. Collection paths in `index.yml` are machine-specific; run the destination's `qmd update` so its local path is synchronized into the SQLite collection metadata.

# Verification

A GPU-generated baseline was copied to CPU-capable destinations. Each destination ran local `qmd update`, verified document/vector counts, and completed a CPU semantic retrieval query successfully.

The destination SQLite checksum may change after `qmd update` because local collection metadata and timestamps are synchronized. Validate document/vector counts and retrieval, not byte-identical database hashes.

# Operating policy

Use snapshot distribution as a bulk-ingestion accelerator, not as the Git synchronization layer. GitHub synchronizes canonical Markdown; each machine keeps an independent local QMD database. After later Markdown changes, either distribute a new quiescent baseline or let each machine run `qmd update` and embed only its local delta.

If the GGUF bytes change while the URI stays the same, force a full re-embed. Never commit QMD SQLite files, model caches, WAL/SHM files, or machine-local QMD state to `agent-memory`.
