---
title: DVLab monitoring dashboard deploy and PR review
scope: projects/dvlab-mis
status: active
updated: 2026-11-19
---

The monitoring dashboard repo is `DVLab-NTU/dvlab-dashboard`. Production is nginx on zeus (`http://192.168.1.201/`, LAN/VPN only), serving the checkout at `/opt/dvlab-dashboard`. That checkout is owned by `swear02`. Deploy a merged change with `ssh su_zeus 'cd /opt/dvlab-dashboard && git pull --ff-only'`. Plain `ssh zeus` logs in as `swear01`: git refuses with "detected dubious ownership", and `swear01` has no sudo on zeus. Do not add a global `safe.directory` and pull as `swear01`, because the files would end up with the wrong owner. Static files take effect immediately; no nginx restart is needed unless `config.env` or the nginx template changed.

The user expects a merged dashboard fix to be deployed and checked on the live URL, not just merged.

PR review: only Codex (`@codex review`) responds on this repo. Swear Review and Gemini were triggered on PRs #9 and #12 and never answered, and the repo has no Claude workflow and no CI. Codex posts a receipt comment that names the reviewed commit; check it against the PR head before merging.

For a local preview, download `usage.json`, `detail.json`, `localusage.json` and `overview.json` from the production URL into `web/` (they are gitignored) and serve `web/` with `python3 -m http.server`.

UI notes from PR #12: the storage page aligns each host's Nth disk with CSS subgrid (`grid-row: span 1+maxDisks`, `grid-template-rows: subgrid`, spacing in padding so empty tracks collapse). Do not put `min-width` on the per-owner bar segments: with 30–40 owners, the forced pixels pushed the owner bar up to 42px past the disk bar. The overview and storage pages share their NFS treemap tiles through `nfsTiles()` in `web/common.js`.
