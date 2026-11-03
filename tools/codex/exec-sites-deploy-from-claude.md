---
title: 從 Claude Code 背景呼叫 codex exec 操作 ChatGPT Sites
scope: tools/codex
tool: codex
status: active
updated: 2026-11-04
---

# 從 Claude Code 背景呼叫 codex exec 操作 ChatGPT Sites

- 便宜的溝通模型：`codex exec -m gpt-6-luna --skip-git-repo-check -C <dir> "<prompt>"`（codex-cli 0.165 接受；查詢約 2–3 分鐘、6–8 萬 tokens）。
- **背景執行一定要 `< /dev/null`**。沒關 stdin 時 `codex exec` 只印 `Reading additional input from stdin...` 就無限等待，看起來像「deploy 跑很久」，其實根本沒開始（2026-11-04 白等 40 分鐘）。建議再包 `timeout 1200`。
- Sites 本機沒有 checkout 時：建立資料夾含 `.openai/hosting.json`（`{"static":{"directory":"dist"},"project_id":"appgprj_..."}`）與 `dist/`，`git init` 後 commit，交給 Codex。Codex 會用 `create_source_repository_write_credential` 推到 Sites 的 source repo；遠端較新時它會 `--force-with-lease` 覆寫 `main`，所以要先確認線上版本內容，必要時先備份線上 HTML。
- 公開站台的 `deploy_site_version` 是需要核准的 MCP 呼叫。`~/.codex/config.toml` 是 `approval_policy = "never"`，exec 下會失敗：`MCP tool call requires approval, but approval policy is never`；`save_site_version` 仍會成功，留下可重用的 version id。
- Claude Code auto mode 會擋 `codex exec --dangerously-bypass-approvals-and-sandbox`（Safety Bypass Flag／Create Unsafe Agents），即使使用者口頭核准也一樣。可行做法：把只做 `deploy_site_version <saved version id>` 的指令交給使用者自己在 terminal 跑，或讓使用者在 Codex app 內按核准。

相關：[[windows-x11-eda-guide-site]]
