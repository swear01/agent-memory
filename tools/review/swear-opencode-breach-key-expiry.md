---
title: 2026-10 OpenCode Console 外洩後 Swear Review 全部上游失效
scope: tools/review Swear Review + deepseek-latch-gateway on Oracle
status: diagnosed 2026-10-22; keys not yet rotated; fastify fix PR swear01/swear-review#12 open
updated: 2026-10-22
---

- 症狀：PR check 變成 `5 of 5 selected item(s) failed`（kind ocr, classification provider），之後沒有新 job 只是沒有新 PR；Swear Review、cloudflared、gateway 程序都正常。
- 根因在 gateway 上游：OpenCode Go 帳號 2、3 回 401 `api_key_expired`（OpenCode Console 約 2026-10-17 外洩後 key 被作廢），帳號 1 429 冷卻，Command Code 回 400 insufficient credits。重啟服務無效，需在 OpenCode Console 重建 key 寫進 `~/.config/deepseek-gateway/config.yaml` 再重啟 gateway。
- 快速判斷：`curl 127.0.0.1:35001/status` 看各 endpoint circuitState，並直接打一次 `/v1/chat/completions` 看 `X-Gateway-Active-Endpoint` 與錯誤內文。
- 同日 `npm audit --omit=dev`：fastify <=5.12.4 有 URL auth bypass 等高風險 advisory，PR #12 升到 5.12.7；braces/micromatch GHSA-vfj7-8cjw-p6xm 無修正版本。lockfile 未含 Shai-Hulud 名單套件。
