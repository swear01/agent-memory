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

## 2026-11-19 NGINX 安全更新

Zeus 的 `nginx`、`nginx-common` 已更新至 Ubuntu 26.04 的 `1.28.3-2ubuntu1.13`，由 `resolute-security` 提供。Ubuntu 套件版號是修補依據，`nginx -v` 仍會顯示 1.28.3。Zeus 的實際連線使用 HTTP/1.1 與 TLS 1.3，因此這批公告的 TLS 1.3 拒絕服務修補也適用。見 [USN-9159-1](https://ubuntu.com/security/notices/USN-9159-1)。

更新後核對所有執行中的 NGINX 程序都使用套件的新執行檔，`nginx -t` 與 `dpkg --audit` 通過。Dashboard 的 HTTP、HTTPS 和 JSON，以及 Valkyrie gateway 的 HTTPS 均通過。設定與 systemd drop-in 跟備份一致。備份在 Zeus 的 `/var/backups/dvlab-nginx-20261119`。在此次查核的七台 DVLab 主機與執行中的 Docker 容器中，Zeus 是唯一找到的 NGINX 服務。Inari 主站使用 Caddy。
