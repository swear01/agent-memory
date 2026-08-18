---
title: Unity validation separates open-project isolation from MCP registration
scope: projects/WirePulling
project: WirePulling
tool: Unity MCP
status: needs-review
confidence: high
evidence: >-
  Same-project batchmode failed because the project was already open; a
  temporary copy built successfully, while Unity MCP resources were visible
  but instances reported zero despite a visible Unity window.
created: 2026-08-18
updated: 2026-08-18
tags:
  - unity
  - unity-mcp
  - batchmode
  - editor
  - needs-review
source_refs:
  - codex-b001-wirepulling:019e3703-4c4f-7802-b082-a6cac6c7a8d7
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Boundary

When the WirePulling project is already open in Unity, same-project batchmode can fail because another Editor instance owns the project. Use a disposable project copy outside the open workspace for batch generation, then compare generated assets before applying results.

MCP resource listing is not proof of an Editor connection. In the same session, the MCP instance list repeatedly reported zero instances and scene/console calls returned `No Unity Editor instances found`, although Computer Use showed a Unity window.

Do not claim Play Mode, runtime Console, or visual MCP validation until an instance is registered.
