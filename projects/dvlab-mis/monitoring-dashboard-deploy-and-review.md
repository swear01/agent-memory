---
title: DVLab monitoring dashboard deploy and PR review
scope: projects/dvlab-mis
status: active
updated: 2026-11-19
---

- Dashboard `DVLab-NTU/dvlab-dashboard` 合併後須部署並讀回。以 checkout owner 執行 `ssh su_zeus 'cd /opt/dvlab-dashboard && git pull --ff-only'`，核對 Zeus LAN/VPN URL `http://192.168.1.201/`。一般 SSH 帳號遇 dubious ownership 時，勿用全域 `safe.directory` 跳過，否則檔案可能留下錯 owner。靜態檔案立即生效，改 `config.env` 或 NGINX template 才處理服務設定。
- 查 repo 現行 review/CI。Codex `@codex review` 曾回傳 review commit 收據，須和最新 head 一致。先前 Gemini／Swear Review 未回應，當時無 CI。
- 本機預覽由 production 取 `usage.json`、`detail.json`、`localusage.json`、`overview.json` 放進 gitignored `web/`，在該處 `python3 -m http.server`。
- Storage 用 CSS subgrid 對齊每台第 N 顆磁碟，`grid-row: span 1+maxDisks`、`grid-template-rows: subgrid`，間距放 padding 以收合空 track。Per-owner bar 的 `min-width` 會讓多 owner 的 bar 溢出。Overview／Storage 共用 `web/common.js` 的 `nfsTiles()`。
- NGINX 安全更新應以 Ubuntu 套件版號核對公告，`nginx -v` 只顯示 upstream 版號。核對 master/workers 的執行檔、`nginx -t`、`dpkg --audit`、Dashboard HTTP/HTTPS 與 JSON，以及同機 Valkyrie gateway 的 HTTPS。

[原始紀錄](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/projects/dvlab-mis/monitoring-dashboard-deploy-and-review.md)
