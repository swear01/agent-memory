---
title: Google Photos → HEIC Q70 遷移專案的 pipeline 操作要點
scope: project
project: google-photos-q70-migration
status: active
confidence: high
created: 2026-10-03
updated: 2026-11-28
tags:
  - google-photos
  - takeout
  - heic
  - drive
  - python
---

## 遷移計畫與分類

- **看起來成功，結果卻是 total=0、checked=0。** `sync_anime_drive.py` 看到 `Visual classification held: ... module 'hashlib' has no attribute 'file_digest'` 時，確認目前 shell 的 Python ≥ 3.11 且有 Pillow。已驗證 `uv run --offline --with pillow python pipeline/<script>.py ...`。一般 `python3` 在歷史非登入 shell 曾解析到 3.9。
- 分類有 `anime`、`real` 和 `review`。人工審閱 JSON 要核對 policy、來源 SHA256／path、原狀態、決議、`review_method=assistant_direct_image_inspection`、非空 reason 及時間。缺欄位可能只留一行 `Visual classification held:`。上傳前乾跑 `classifications()`，逐項比對凍結範圍的 SHA 與決議。
- `anime` 進既有 Drive `Media/anime/picture`；`real` 才進 Photos，`review`／`unknown` 排除。Drive 複製回條不能授權刪 Photos 舊件（`anime_photos_cleanup_not_inferred_from_drive_copy`）。工作區本身不使用 Git，`state/` 是計畫與回條的權威來源。復原時查 `state/control-backup-*.zip` 和 Drive 備份；`pipeline/recovery_health.py` 可離線查 worker 與各階段。

## 雲端寫入與清理

- **`Photos web session requires refresh`，或 Photos 首頁 302 到介紹頁。** 先讓持有有效內建瀏覽器工作階段的 session 復原。複製的狀態會過期，瀏覽器 socket 又綁 session 或 turn。此時可以先離線做 `--plan`，不嘗試重送雲端寫入。
- `--plan` 凍結分類、來源／成品 SHA256、Drive 的唯一檔名、size、MD5、file ID 和個別對應，讓授權綁該計畫 SHA。到 `--run` 才依 live 回讀核對 media key、dedup、檔名、位元組與垃圾桶狀態。寫 journal `sending` 後才送出，送完逐項以垃圾桶回讀對帳。結果不明先回查，避免重送。只移入可恢復垃圾桶；原始紀錄與進度備份留在專案。
- **非原畫質副本的內容 SHA 對不上。** 若要用替代身分證據，需另有綁 plan SHA 的授權。核對 Takeout sidecar 的 media key、`original_quality=false`、精確檔名與位元組數，確認未在垃圾桶，並記錄該筆用的證據。重開 journal 前先匯出前次回條。

[完整原始紀錄與逐批驗證](https://github.com/swear01/agent-memory/blob/93786b66aa982bda81d4c1ebe6cfceaa832b5d32/projects/google-photos-q70-migration/pipeline-operations.md)
