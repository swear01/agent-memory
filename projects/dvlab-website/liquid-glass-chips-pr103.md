---
title: DVLab liquid-glass chips PR #103 deploy
scope: projects/dvlab-website
project: DVLab-NTU/dvlab-ntu.github.io
status: active
updated: 2026-10-02
---

## Deployed 2026-10-02 (Asia/Taipei)

- PR #103 squash-merged to `main` as `9aff7ee0c5ae400c85a188621e8a23aa4fc5445a` (short `9aff7ee`).
- Head tip before merge: `1805cc6223c8ee62b63038c61c7d4236c04f7d91` (CI verify enabled/disabled success).
- GitHub Pages workflow `36984210862` on that merge commit **success**; backup https://dvlab-ntu.github.io/ serves glass UI (`/_astro/awards.DrtS40R4.css`).
- inari primary https://dvlab.ee.ntu.edu.tw/ : `current` → `releases/20261002-9aff7ee` (previous `releases/20261001-2d370c3` kept). Deployed same Pages `github-pages` artifact (`artifact.tar` SHA-256 `8e8dc0928f6994cf89eb7f6479c0f1e5189a4c3a2a2716f2af1960971945e0d7`); no Caddy restart. Deploy path: Mac SwairM5 (`e02ab314-a679-4675-87b6-1d51914ed74f`) SSH Host `inari`.
- Live spot-check: home news `glass-chip` date chips; papers `glass-list-card` / meta `DATE · YEAR`; paper detail labeled **會議/Venue** + **年份/Year** chips; awards glass cards; `has-unified-page-backdrop` + unified footer. Primary and Pages CSS hash match `f37998d67351fc4d7dd4ea4dd0e457f9e5671e2195a94092173e73a491c34d87`.

## Design prefs locked in #103

- Site-wide liquid-glass translucency via `--glass-bg*` tokens (approved levels); chips use `--glass-bg-chip*`; paper **list** rows use denser `--glass-bg-panel`.
- Home `.home-news-item` vertically centers date chip with title (`align-items: center`).
- Paper detail: labeled chips only (會議/Venue year-stripped + 年份/Year); BibTeX + copy on one `btn-ghost` row; no Links/back buttons.
- Single `SiteFooter.astro` via `BaseLayout`; short pages keep footer at viewport bottom with flex column + `main` flex-grow; transparent footer on backdrop.

## Obsolete

- PR body “Draft — not merged / not deployed” is obsolete after this production deploy.
