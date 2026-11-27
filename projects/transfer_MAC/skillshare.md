---
title: skillshare is the canonical skill source while MCP remains separate
scope: projects/transfer_MAC
project: transfer_MAC
tool: skillshare
status: active
confidence: high
evidence: Successful target sync, nested-skill naming fix, explicit migration precedence, and separate MCP workflow; legacy overlap remains a verification caveat.
created: 2026-08-18
updated: 2026-11-18
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

- **同名 skill 在不同 agents 載入不同版本。** transfer_MAC 的正式來源是 `.skillshare/skills/`，設定由 `stow/config/.config/skillshare/config.yaml` 管理。用 `skillshare status --json` 查各台 source/target，核對 link 與內容；`local` 項先比對備份，現有 canonical 的重複副本才改 link，獨有項審查後再導入。Nested skills 用 `target_naming: flat`；`skillshare doctor` 只能確認所有權，外部 skills 另做行為及憑證檢查，不移 manifest 或 plugin 自有檔。
- **Mac detached skill submodule 出現 unpushable merge commit。** 別在此跑 `skillshare pull`。先保留未推 commits 至分支，在新 upstream worktree cherry-pick／正常驗證及 PR。上游合併後在 Mac fetch、detach-checkout exact merge SHA、核對 clean 再 `skillshare sync`；父 gitlink 分開更新。Git 即使 deinit 仍可能拒絕移除有 submodule 的乾淨工作樹，保留並報原因。
- **Gemini CLI 清理把 Antigravity 或 MCP 也移了。** AGY 的 `~/.gemini/skills` 可以是現行 target；`GEMINI.md` 是指令連結，MCP 由 canonical manifest 和 `scripts/sync-ai-agent-configs.py` 另渲染。HAPI transcript 恢復與本機 AGY 登入狀態相互獨立。Grok 已讀 Claude/universal targets；加 Grok target 造成 overlap。移除 target 可能把連結實體化留舊副本，須確認歸屬再處理。
- **HAPI 說 skill 是新版，但 agent 仍讀舊 copy。** `hapi-session-control` 0.35 起從 shared-skills / Skillshare 發布，舊 `.hapi-managed` 副本可能遮住。先核 app runtime、共享 commit、每台實際 link 與 file content，不能以 PR 文件更新推定已部署。接口改動以 [0.35 peer 指令](../../tools/hapi/peer-cli-0-35-command-changes.md) 為準。

[原始紀錄](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/projects/transfer_MAC/skillshare.md)
