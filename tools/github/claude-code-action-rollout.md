---
title: Claude Code GitHub Action 多 repo 佈署
scope: tools/github
tags: [claude-code-action, github-app, secrets, ruleset, pr-review]
status: active
updated: 2026-10-21
---

2026-10 起 PR review 備援順序改為 Gemini → Claude（`@claude review this PR`）→ Cursor Bugbot，取代 Codex；細節在 `github-pr-review-loop` skill。Codex 在多數 repo 本來就沒設定 cloud environment，留言只會回「create an environment」。

讓 `@claude` 在某個 repo 真的運作需要三件事，缺一不可：

1. Claude GitHub App 安裝並涵蓋該 repo（https://github.com/apps/claude，選 All repositories）。這是使用者本人的瀏覽器授權，agent 不代做。
2. repo 內有 `.github/workflows/claude.yml`，使用 `anthropics/claude-code-action@v1`，job 用 `if: contains(..., '@claude')` 篩選，只在提及時跑；permissions 需含 `id-token: write`（App 的 OIDC）。
3. repo secret `CLAUDE_CODE_OAUTH_TOKEN`（由 `claude setup-token` 產生，走訂閱額度）或 `ANTHROPIC_API_KEY`。token 由使用者自己設定，agent 不輸入憑證。

觀察到的限制：

- 個人帳號沒有 account-level Actions secret，只能每個 repo 各設一次；org（例如 DVLab-NTU）可用 `gh secret set --org <org> --visibility all` 一次套用。
- `gh api user/installations` 用 `gh` 的 OAuth token 會回 403（需要 GitHub App 的 user access token），無法用 CLI 確認 App 是否安裝；請使用者看 https://github.com/settings/installations 。
- 有 ruleset 要求 `Swear Review` status check 的 repo（例如 `swear01/swear-review-e2e`），用 contents API 直接寫 main 會回 409 `Required status check "Swear Review" is expected.`，必須改走 branch + PR。
- 推 workflow 檔需要 `gh` token 具 `workflow` scope。
- fork 預設停用 Actions；重新啟用後所有上游 workflow 一起生效，需逐一檢查哪些依賴 fork 沒有的 secret 或會對外發佈（例如 ghcr image push）。
