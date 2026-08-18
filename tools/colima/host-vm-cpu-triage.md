---
title: Triage virtualization CPU through host, guest, and container layers
scope: tools/colima
tool: Colima
status: active
confidence: high
evidence: >-
  Host, guest, and container measurements consistently isolated the CPU usage
  to guest systemd-logind rather than the long-running container.
created: 2026-08-18
updated: 2026-08-18
tags:
  - colima
  - lima
  - virtualization
  - docker
  - diagnostics
source_refs:
  - codex-b001-linked-versedx:019ef36a-e21b-7060-884c-74ac3729fbf7
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Procedure

The visible macOS Virtualization.framework process is only the VM wrapper. Attribute CPU in layers:

1. Use host process/file inspection to confirm the VM belongs to Colima.
2. Check `colima status` and sample guest processes through `colima ssh`.
3. Check `docker stats` and `docker top` before blaming a container.
4. Inspect guest systemd/D-Bus health when container CPU is low.

A standalone `limactl list` result may not be authoritative while Colima is managing the VM; compare it with `colima status` and guest evidence.
