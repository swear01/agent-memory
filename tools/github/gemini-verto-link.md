---
title: Gemini Code Assist 沒有審查 repo，直到執行 gemini-link
scope: tools/github; Gemini Code Assist; swear01/Verto
status: active
updated: 2026-10-25
---

# Gemini Code Assist never reviewed a repo until `gemini-link` ran

Evidence (2026-10-25): swear01/Verto PRs #1–#31 had zero `gemini-code-assist[bot]` comments.
The repo was simply not linked to Gemini (Developer Connect link missing). The GitHub app itself
was fine for the account.

Fix: `gemini-link swear01/Verto` (prints `linked`). Then `/gemini review` on PR #30 (already
merged) got a real review in ~2 minutes (review 5644383287), so closed/merged PRs can be reviewed too.

Lesson: for any repo with no Gemini output, run `gemini-link OWNER/REPO` first; no browser step needed.
