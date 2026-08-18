---
title: skillshare is the canonical skill source while MCP remains separate
scope: projects/transfer_MAC
project: transfer_MAC
tool: skillshare
status: active
confidence: high
evidence: Successful target sync, nested-skill naming fix, explicit migration precedence, and separate MCP workflow; legacy overlap remains a verification caveat.
created: 2026-08-18
updated: 2026-08-18
tags:
  - skillshare
  - skills
  - mcp
  - transfer-mac
source_refs:
  - pilot-hapi-export:seq=229
  - pilot-hapi-export:seq=235-239
  - pilot-hapi-export:seq=257
  - pilot-hapi-export:seq=261
  - pilot-hapi-export:seq=269
  - pilot-hapi-export:seq=273
  - pilot-hapi-export:seq=277
generated_by: openai-codex/gpt-5.6-luna
redaction: passed
---

# Ownership

Use `.skillshare/skills/` as the repository skill source and manage its configuration through `stow/config/.config/skillshare/config.yaml`.

`skillshare` owns skill targets such as:

- `~/.agents/skills`
- `~/.claude/skills`
- `~/.cursor/skills`
- `~/.config/opencode/skills`
- `~/.gemini/skills`

Do not use GNU Stow or the MCP renderer to manage those skill directories.

# Nested skills

Use `target_naming: flat` when a source contains nested skills. Standard naming skipped nested `socv-ln-crawler` skills; flat naming exposed them as distinct target names and produced a complete merged sync.

# Migration precedence

During the explicit migration, the live machine skill source won over divergent older Stow/repository copies. Keep one canonical source rather than merging two same-named versions indefinitely.

# MCP separation

skillshare handles skills only. MCP remains in the canonical MCP configuration and is rendered separately by `scripts/sync-ai-agent-configs.py`.

# Import and audit boundary

Review external and plugin-managed skill candidates before importing them. A skillshare behavioral-risk audit is not the same as a credential scan: run both, and do not copy machine-local configuration containing secrets into the skill source.

The pilot still showed legacy target-overlap warnings; do not claim ownership is clean until `skillshare doctor` confirms that legacy writers no longer manage skill targets.
