---
title: Claude 桌面版設定頁的 Skills 不會列出 skillshare 同步到本機的 skill
scope: tools/claude-desktop
tool: Claude desktop app
machine: SwairM5
status: active
confidence: high
evidence: 2026-10-19 使用者截圖顯示設定頁只有「From Anthropic & Partners」與「From marketplaces you added」；同時 Code 分頁 session 確實載入了 `~/.claude/skills` 裡的 skillshare 連結。anthropics/claude-code issues #31597、#39994、#43095 回報同一現象，多數被關閉但沒有修正。
created: 2026-10-19
updated: 2026-10-19
tags:
  - claude-desktop
  - skillshare
  - skills
---

# 設定頁 Skills 與本機 skill 是兩套

Claude 桌面版的 Settings → Customize → Skills 列的是帳號層級的 skill，例如 Anthropic、partner 與 marketplace plugin 提供的，不會掃描本機的 `~/.claude/skills`。所以 skillshare 用 symlink 同步過去的 skill 不會出現在這一頁，但 Claude Code（CLI 與桌面版 Code 分頁）仍會正常載入。判斷 skillshare 有沒有生效，要看 session 的可用 skill 清單，或 `skillshare status` 裡 claude target 的 `shared` 數，不要看設定頁。

同步方向是帳號到本機：帳號 skill 會被寫進 `~/.claude/skills/synced/<session-id>/`（例如 google-workspace、deep-research、docx），反過來不會發生。這是已知的問題（anthropics/claude-code #31597、#39994、#43095）：面板不會列出 `~/.claude/skills` 裡的個人 skill，但在 `/` 選單和對話中都能正常使用。有回報說，Windows 上面板只會列出部分 skill，而且每次打開結果不同。「+ Add」據回報只接受單一檔案，沒辦法註冊多檔案的 skill 資料夾。就算上傳成功，也只是複製一份，不會跟著 skillshare 更新，而且可能和本機同名 skill 重複。

在 Code 分頁裡看不到某個 skillshare skill 時，先排除這些刻意設計的原因：

- source 的 `.skillignore` 寫了 `shelf/**`：shelf 裡的 skill 不同步，要用 `skillshare-reveal` 個別連結到專案。
- claude target 的 `exclude`（例如 `ponytail*`，因為已經由 plugin 提供）。
- frontmatter 有 `disable-model-invocation: true`：只能用 `/name` 手動呼叫，不會出現在模型的清單裡。
- 資料夾裡沒有 `SKILL.md`：`doctor` 會回報 `skills_validity`。

computer-use 不能操作 Claude 桌面版自己的視窗（`request_access` 會直接拒絕），看不到設定頁時，要請使用者截圖或捲動。
