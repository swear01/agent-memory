---
title: Migrated MCP launchers should resolve dependencies at runtime
scope: tools/mcp
status: draft
confidence: medium
evidence: Portable launcher template was created and inspected; runtime launch was not exercised in the pilot.
created: 2026-08-18
updated: 2026-08-18
tags:
  - mcp
  - migration
  - uvx
  - portability
source_refs:
  - pilot-cursor:cursor.jsonl:5
  - pilot-cursor:cursor.jsonl:8
  - pilot-cursor:cursor.jsonl:21
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Launcher rule

Migrated MCP configuration should invoke a runtime-resolved executable such as `uvx` with an explicit package/version and entry point. Do not preserve an old machine's absolute user path.

# Credential boundary

Keep OAuth state, tokens, and browser credentials outside the portable MCP template. Reauthorize them on the destination machine.

# Verification

A valid template proves only that the configuration artifact is portable. Mark the MCP runtime verified only after rendering the native config and successfully launching the server.
