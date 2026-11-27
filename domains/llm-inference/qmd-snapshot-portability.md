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

# Reuse a quiescent QMD snapshot across machines

- GPU-generated vectors work on CPU destinations. The current fleet uses the same embeddinggemma-300M model URI and bytes. Match QMD and sqlite-vec compatibility and the Git content baseline. Copy the model file when needed, rather than platform-specific runtime files. Any change to bytes behind an unchanged model URI requires new embeddings.
- Checkpoint and close the source database before copying. For a clean checkpoint there should be no pending WAL contents; if sidecars still carry data, establish a coherent snapshot first. Never copy only a live main SQLite file or mix its WAL/SHM with another index. Stop the destination's QMD writers and keep a recoverable old index set before replacement. Live databases and SQLite sidecars are local state.
- Confirm that the snapshot contains only collections approved for the destination. The Mac's general index may include local notes beyond shared memory, even when the command used to produce embeddings specified `-c memory`. Check the collection inventory or build a separate shared-only index before transfer.
- Keep destination-specific config and cache directory, update the destination collection paths, and verify with counts, indexed path probes and a semantic canary. A changed SQLite checksum after update is normal because local metadata may differ; integrity check or counts alone do not establish retrieval. Git retains the canonical notes; snapshots accelerate bulk embeddings and stay out of Git. See [Oracle CPU limitations](../../machines/oracle/qmd-vsearch-slow.md).

[整理前的測量與逐次驗證紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/domains/llm-inference/qmd-snapshot-portability.md)
