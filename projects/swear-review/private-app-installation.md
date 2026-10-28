---
title: 私人帳號擁有的 private GitHub App 不能直接安裝到組織
scope: projects/swear-review GitHub App installation
status: verified 2026-10-28; no App visibility changes
updated: 2026-10-28
---

- Swear Review App 由個人帳號擁有，且 registration 為 private。正式 App API 的 installation 清單只有擁有者個人帳號；目標組織 repository 的 installation 查詢回 404。
- GitHub 官方規則：private App 只能安裝到擁有 App 的帳號。即使使用者同時是目標組織 admin，也不能直接跨帳號安裝；安裝網址加 `target_id` 不會解除限制。先前「org admin 可直接安裝」的推論不完整。
- 將 registration 改 public 可讓其他帳號安裝，但也讓任何 GitHub 使用者可以安裝；不必上 Marketplace。這是服務對外範圍變更，不能當成只加一個 repository 的操作。
- 本次查正式 scheduler 與 config schema，沒有 owner／installation allowlist。`review.auto=false` 只關閉自動 review；新安裝者只要有自己 repository 的 write／maintain／admin 權限，仍可用 `/swear-review` 消耗共用 LLM 額度。
- 若要服務指定組織，先決定 public App 配合安裝範圍限制，或獨立的組織擁有 private App；本次未改 visibility、App 設定或部署。
- 官方來源：GitHub Docs，`apps/creating-github-apps/registering-a-github-app/making-a-github-app-public-or-private`。
