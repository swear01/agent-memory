---
title: 從本機紀錄統計各 agent（Claude、Codex、agy、Cursor）的 token 用量
scope: tools/llm
status: active
confidence: medium
evidence: 2026-11-21 為 CVSD HW4 簡報統計 11/11–11/20 各階段、各模型 token；Claude 與 Codex 欄位為官方 JSON，agy 欄位由 protobuf 反推（兩筆呼叫對照 output = thinking + visible 吻合）。
created: 2026-11-21
updated: 2026-11-21
tags:
  - token
  - usage
  - codex
  - antigravity
  - claude-code
---

# 從本機紀錄統計各 agent 的 token 用量

搜尋關鍵字：token 用量、token 統計、usage、cache read、codex token_count、agy 用量、Gemini token。

用 output tokens 比較工作量：input 大多是 prompt cache 讀取（Claude 約 95%、Codex 約 97%），總量會被放大。

- **Claude Code**（含 `claude -p` 與 subagent）：`~/.claude/projects/<cwd>/**/*.jsonl`（含 `<session>/subagents/`）。每行 `message.usage` 有 `input_tokens`、`cache_creation_input_tokens`、`cache_read_input_tokens`、`output_tokens`；以 `(message.id, requestId)` 去重，`message.model` 區分 Opus／Sonnet／Fable。用行內 `timestamp`（UTC）切時間段。
- **Codex**：`~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`。`session_meta.payload.cwd` 判斷專案，`turn_context.payload.model` 是模型；`event_msg` 的 `payload.type == "token_count"` 裡 `info.last_token_usage` 是單次呼叫（可依時間分段加總），`info.total_token_usage` 是整個 session 累計（`input_tokens`、`cached_input_tokens`、`output_tokens`、`reasoning_output_tokens`）。
- **Antigravity CLI（agy，Gemini）**：`~/.gemini/antigravity-cli/conversations/*.db`（SQLite）。`gen_metadata.data` 是 protobuf：欄位 1 → 4 是 usage，其中 2 = 未快取 input、5 = 快取 input、3 = output（= 9 thinking + 10 可見）；1 → 19 是模型名（如 `gemini-4-neon`）。沒有逐筆時間，用 `.db` 的 mtime 歸段。用 `mode=ro` 開。
- **Cursor Agent（Grok 等）**：`~/.cursor/chats/<hash>/<id>/store.db` 內有 JSON 片段 `"inputTokens"`、`"outputTokens"`；經 HAPI 在遠端機器跑的 session 本機沒有紀錄。

依專案過濾時，以檔案內文出現專案關鍵字的次數判斷（例如 `hw4` vs `hw3`），重疊期間要分清楚。
