---
title: "Cursor model picker 缺新模型：清掉 runner 的 cursor-models.json"
scope: tools/hapi
status: active
updated: 2026-11-19
---

# Cursor model picker 缺新模型：清掉 runner 的 cursor-models.json

症狀：HAPI Web 某台機器的 Cursor model 選單少了新模型（2026-11-19 是 Gemini 4 Neon），但該機 `agent models` CLI 已列出，且各機 cursor-agent 版本相同。

原因不是 CLI 版本，是 runner 的快取 `$HAPI_HOME/cache/cursor-models.json`（Mac/oracle/swop 在 `~/.hapi/cache/`，Linux 在 `/var/tmp/hapi-<machine>/cache/`）。重新查詢只會更新 mtime，內容仍是舊清單，所以看 mtime 判斷不出它舊了。

處理：把該檔 rename 成 `.bak-<date>`，再打 `GET /api/machines/<id>/cursor-models`，runner 會重抓並寫入新清單。不用重啟 runner，也不影響 session。驗證方式：回應的 `availableModels` 裡要有新的 `modelId`。

先用 `grep -q <new-model-id>` 檢查快取有沒有新模型，再決定要不要清。不要從 CLI 版本或檔案 mtime 推論清單是新的。
