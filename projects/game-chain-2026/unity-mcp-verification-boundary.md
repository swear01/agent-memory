---
title: Unity MCP has separate package, server, and Roslyn verification layers
scope: projects/game-chain-2026
project: game-chain-2026
tool: Unity MCP
status: needs-review
confidence: medium
evidence: >-
  The Unity package was pinned locally to v9.7.1, the Python server help
  resolved, and five Roslyn DLLs were downloaded, but the requested Unity
  editor was unavailable and no runtime handshake was verified.
created: 2026-08-18
updated: 2026-08-18
tags:
  - unity
  - unity-mcp
  - roslyn
  - uvx
  - needs-review
source_refs:
  - codex-b001-game-chain:019e65ab-75f3-75e2-b359-b0b5bf06d5ed
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Layers

For this Unity project, keep these verification layers separate:

- Unity package: `com.coplaydev.unity-mcp` pinned to `v9.7.1`.
- Python server: independently checked with `uvx` help from the matching server source.
- Optional Roslyn editor support: `Microsoft.CodeAnalysis.Common` and `Microsoft.CodeAnalysis.CSharp` 4.12.0, `System.Collections.Immutable` and `System.Reflection.Metadata` 8.0.0, and `System.Runtime.CompilerServices.Unsafe` 6.0.0.

Local `skip-worktree` and exclude rules only show that package changes are hidden from normal Git status. They do not prove package import or runtime availability.

The requested Unity editor was unavailable during this investigation. Keep package, server, Roslyn, and Unity handshake status separate until Editor validation succeeds.
