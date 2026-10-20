---
title: Claude Code auto mode 會擋下的 GitHub 寫入操作
scope: tools/claude-code
tags: [auto-mode, classifier, gh, github]
status: active
updated: 2026-10-21
---

2026-10 在 Claude Code desktop auto mode 實測，下列 `gh` 操作即使使用者在對話中明確要求，也會被 auto mode classifier 拒絕：

- `gh pr merge <n> --merge --delete-branch`：理由 `Merge Without Review`。PR 上即使已有 Gemini 對最新 head 的乾淨 review、CI 全綠，classifier 也不會參考。
- `gh workflow disable <file> -R <fork>`：理由 `CI Bypass`，即使目標只是停掉 fork 上缺 secret、必定失敗的上游 workflow。
- 一次對多個 repo 的預設分支做 `gh api -X PUT repos/<repo>/contents/.github/workflows/<file>` 迴圈：被拒且沒有給理由。
- 呼叫 `update-config` skill 為自己在 `~/.claude/settings.json` 加 allow rule：理由 `Self-Modification`。

被拒後不要改寫成等效指令繞過。可行做法：把完整、可直接執行的單行指令交給使用者自己跑，或請使用者在 Claude Code permission 設定中針對該指令前綴加 allow rule 後再重跑。使用者離開 auto mode（一般或 bypass permissions 模式）後，同樣的 `gh pr merge` 與使用者明確要求的 settings 編輯即可執行；auto mode 下的 classifier 不看 PR 是否已有 review。其他不依賴該步驟的工作照常完成。
