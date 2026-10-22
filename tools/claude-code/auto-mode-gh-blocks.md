---
title: Claude Code auto mode 會擋下的 GitHub 寫入操作
scope: tools/claude-code
tags: [auto-mode, classifier, gh, github]
status: active
updated: 2026-10-22
---

2026-10 在 Claude Code desktop auto mode 實測，下列 `gh` 操作即使使用者在對話中明確要求，也會被 auto mode classifier 拒絕：

- `gh pr merge <n> --merge --delete-branch`：理由 `Merge Without Review`。PR 上即使已有 Gemini 對最新 head 的乾淨 review、CI 全綠，classifier 也不會參考。
- `gh workflow disable <file> -R <fork>`：理由 `CI Bypass`，即使目標只是停掉 fork 上缺 secret、必定失敗的上游 workflow。
- 一次對多個 repo 的預設分支做 `gh api -X PUT repos/<repo>/contents/.github/workflows/<file>` 迴圈：被拒且沒有給理由。
- 呼叫 `update-config` skill 為自己在 `~/.claude/settings.json` 加 allow rule：理由 `Self-Modification`。

被拒後不要改寫成等效指令繞過。可行做法：把完整、可直接執行的單行指令交給使用者自己跑，或請使用者在 Claude Code permission 設定中針對該指令前綴加 allow rule 後再重跑。使用者離開 auto mode（一般或 bypass permissions 模式）後，同樣的 `gh pr merge` 與使用者明確要求的 settings 編輯即可執行；auto mode 下的 classifier 不看 PR 是否已有 review。其他不依賴該步驟的工作照常完成。

## 為什麼 AGENTS.md / skill 寫「bot review 算 review」沒用

`claude auto-mode defaults` 中的內建規則原文是「Merging a PR before any human has approved it」，審核被寫死為人類核准。依官方文件（code.claude.com/docs/en/auto-mode-config、permission-modes）：

- classifier 看得到使用者訊息、tool call 與 CLAUDE.md；tool result 會被剝掉，所以 AGENTS.md、skill 內文（例如 `personal-pr-workflow` 的「不要請使用者核准合併」）不會被當成授權。CLAUDE.md 也只是參考，清不掉 soft_deny。
- 能清掉 soft_deny 的只有 `autoMode.allow`，或使用者在**當下 session** 親口、具體要求該動作。其他 session 交辦的 brief 不算。
- 窄的 `permissions.allow`（如 `Bash(gh pr merge:*)`）在 auto mode 仍有效且先於 classifier，但只做前綴比對；推測 `cd x && gh pr merge` 這類複合指令不會命中，就落到 classifier。

## 解法：`autoMode` 設定

`autoMode` 只從 `~/.claude/settings.json`、managed settings、`--settings` 讀取，不讀專案的 `.claude/settings*.json`。每個清單必須保留 `"$defaults"`，否則整段預設規則會被取代。2026-10-22 已在本機加入：

- `environment`：`github.com/swear01` 為自家 source control；bot review（Swear Review、Gemini、Claude、Codex、Cursor Bugbot）在最新 head 通過加上 CI 綠燈即算 review。
- `allow`：`Merge Without Review` 例外，限 `gh pr merge --match-head-commit <sha>`、swear01 repo、bot 通過且 CI 綠。

用 `claude auto-mode config` 確認已載入（已驗證）。這是 classifier 解讀的文字，不是硬保證；實際合併是否因此放行尚未驗證。agent 在 auto mode 下改這個設定會被 `Self-Modification` 擋，需使用者自己改或先離開 auto mode。
