---
title: QMD vectors can be distributed as quiescent local snapshots
scope: domains/llm-inference
tool: qmd
status: active
confidence: high
evidence: QMD 2.8.3 source review plus a GPU-to-CPU snapshot pilot across the fleet.
created: 2026-08-18
updated: 2026-08-18
tags:
  - qmd
  - embeddings
  - sqlite
  - gpu
  - cpu
  - fleet
source_refs:
  - qmd-portability-review:QMD-2.8.3
  - qmd-snapshot-pilot:athena-to-oracle
  - qmd-snapshot-pilot:athena-to-zeus
  - qmd-snapshot-pilot:athena-to-nfs-fleet
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# What can be shared

The Qwen3-Embedding-0.6B Q8_0 GGUF model can be copied between machines when the file checksum matches. Copy the model file, not native `node_modules` or platform-specific runtime binaries.

A completed QMD SQLite index can also be copied as a **quiescent local snapshot**. QMD vectors are stored in SQLite/sqlite-vec and do not contain GPU, CUDA, Vulkan, Metal, or VRAM state. A CPU machine can search vectors produced on a GPU machine.

# Snapshot requirements

Only snapshot after all QMD processes have exited. If `index.sqlite-wal` or `index.sqlite-shm` exists, treat the main file and sidecars as one coherent snapshot; never copy only the main file from a live WAL database. Do not put one live QMD SQLite database on NFS or sync multiple writers into it.

The destination needs the same QMD/sqlite-vec compatibility, the exact embedding model URI and model bytes, and a matching Markdown corpus. Collection paths in `index.yml` are machine-specific; run the destination's `qmd update` so its local path is synchronized into the SQLite collection metadata.

# Verification

A GPU baseline containing 13 Markdown documents and 13 vectors was copied from Athena to the NFS fleet, Zeus, and Oracle. Each destination reported 13 indexed documents and 13 embedded vectors after local `qmd update`; `qmd embed` found no missing embeddings. Oracle's CPU-only structured semantic query returned the copied memory successfully.

The destination SQLite checksum may change after `qmd update` because local collection metadata and timestamps are synchronized. Validate document/vector counts and retrieval, not byte-identical database hashes.

# Operating policy

Use snapshot distribution as a bulk-ingestion accelerator, not as the Git synchronization layer. GitHub synchronizes canonical Markdown; each machine keeps an independent local QMD database. After later Markdown changes, either distribute a new quiescent baseline or let each machine run `qmd update` and embed only its local delta.

If the GGUF bytes change while the URI stays the same, force a full re-embed. Never commit QMD SQLite files, model caches, WAL/SHM files, or machine-local QMD state to `agent-memory`.
