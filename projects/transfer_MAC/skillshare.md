---
title: skillshare is the canonical skill source while MCP remains separate
scope: projects/transfer_MAC
project: transfer_MAC
tool: skillshare
status: active
confidence: high
evidence: Successful target sync, nested-skill naming fix, explicit migration precedence, and separate MCP workflow; legacy overlap remains a verification caveat.
created: 2026-08-18
updated: 2026-09-06
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

# Submodule update boundary

The active Mac source is the detached `.skillshare/skills` submodule. Running `skillshare pull` there merges `origin/main` into detached HEAD and can create local, unpushable merge commits even when the resulting tree equals upstream. After a shared-skills PR merges, fetch and detach-checkout the exact upstream merge SHA, verify a clean source, then run `skillshare sync`; update the parent gitlink separately. Do not use `skillshare pull` for this detached submodule checkout.

# Worktree cleanup boundary

On the installed Git version, `git worktree remove` refuses a clean worktree whose tree contains the tracked `.skillshare/skills` submodule, even after `git submodule deinit`. The workflow policy forbids `git worktree remove --force`, so keep and report that worktree rather than bypassing the safety guard. This does not indicate dirty task files.

# MCP separation

skillshare handles skills only. MCP remains in the canonical MCP configuration and is rendered separately by `scripts/sync-ai-agent-configs.py`.

# Import and audit boundary

Review external and plugin-managed skill candidates before importing them. A skillshare behavioral-risk audit is not the same as a credential scan: run both, and do not copy machine-local configuration containing secrets into the skill source.

The pilot still showed legacy target-overlap warnings; do not claim ownership is clean until `skillshare doctor` confirms that legacy writers no longer manage skill targets.

# Gemini CLI scope correction and Antigravity restoration

On 2026-08-22, a cleanup that was intended for Gemini CLI was corrected so Antigravity/AGY remained managed. The shared-skills source was restored and fleet verification confirmed `agy 1.1.18`, the `antigravity` target at `~/.gemini/skills`, six active skillshare targets, and no sync drift. The target-overlap warning is expected for this multi-runtime layout.

`AGENTS.md` is not a Skillshare skill: the canonical file remains under the agent configuration source, and `GEMINI.md`, Codex, Claude, OpenCode, and Pi instruction files are targeted links to it. Do not interpret `GEMINI.md` or the Antigravity target as Gemini CLI residue. The four shared MCP servers remain separate from Skillshare and are rendered from the canonical MCP manifest.

HAPI AGY session rows and transcripts are independent of local AGY authentication state. Restoring HAPI history does not restore local login credentials; after reinstalling or rebuilding Antigravity, AGY may require a fresh login. An operator intentionally closing an AGY process does not mean its preserved HAPI session data was deleted.

## Canonical links and HAPI session-control alignment

所有已由 Skillshare 管理的 Agent 技能都應連到正式來源；不能只修 Claude，也不能保留同名的獨立舊副本。先以 `skillshare status --json` 找出每台 active source 與 targets，逐一檢查連結及內容。`sync` 的 `local` 計數代表仍有獨立項目；先比對、備份，再將正式來源已有的同名副本改為符號連結。沒有正式來源的獨有技能要先審查導入，不能直接刪除。不要移動 Skillshare manifest 或 plugin 自有技能。

HAPI PR #1771 的 `cli/skills/hapi-session-control/SKILL.md` 是該功能的正式指引。以 PR 最新提交比對，而非以舊的全域副本反推功能缺失。CLI 流程為 `spawn-peer` → `wait-peer` → `archive-peer`；父 Agent 負責收集結果後清理自建子 session，等待失敗時也要清理。`abort-peer` 中斷回合、`stop-peer` 停止程序、`archive-peer` 停止並歸檔；舊的 `stop-peer --archive` 語法不適用。CLI 操作不需要 MCP。

PR 文件、共享來源、Agent target 及已安裝 HAPI 版本是四個不同層次。文件同步不能當作 binary 已升級；發布 shared-skills 也不能當作每台機器已 pull。遠端無法連線時明確列出未驗證範圍。更新記憶後刷新本機 QMD 並查詢驗證。
