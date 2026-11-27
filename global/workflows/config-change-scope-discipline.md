---
title: 設定替換後用集合差異與實際呼叫驗證範圍
scope: global/workflows
status: active
updated: 2026-11-28
created: 2026-09-12
---

- **只改模型名，其他模型卻消失。** Rename／replace 只修改舊 ID 對應項目。用設定集合 diff 驗證舊名稱換成新名稱，再用實際 runtime registry 確認其餘成員還在。Pi 的 HAPI picker 由 `strict-model-allowlist.ts` 的 runtime registry 控制，還要核對 TUI 的 `enabledModels` 與 `model-filter.json`。先驗證單機再推 fleet。[Pi 模型遷移](../../tools/pi/deepseek-model-id-migration.md)
- **替換依賴，卻停掉它的應用程式。** 逐一核對替換元件、保留的呼叫端，以及獲授權停用的服務。替換 Swear Review 的 gateway 時，先讓 Review 實際經過新 gateway 完成一輪，再停止明確要替換的舊元件。單看新 gateway 的 health 或模型清單會漏掉 caller 失效。[Swear Review 案例](../../projects/deepseek-latch-gateway/cliproxyapi-retirement.md)
- 相似名稱或轉錄使停用對象含糊時，先完成可逆的查證；只釐清會中斷服務或隱藏資料的動作。回報應各有證據：指定項目已改、原有其餘成員和呼叫端還可用。

[原始診斷與查證紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/global/workflows/config-change-scope-discipline.md)
