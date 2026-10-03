---
title: Google Photos → HEIC Q70 遷移專案的 pipeline 操作要點
scope: project
project: google-photos-q70-migration
status: active
confidence: high
created: 2026-10-03
updated: 2026-10-03
tags:
  - google-photos
  - takeout
  - heic
  - drive
  - python
---

# Google Photos → HEIC Q70 遷移專案的 pipeline 操作要點

## 專案位置與性質

- 工作區：`<project-root>`（= `<home>/Documents/Codex/2026-09-06/google-drive-api-thread-01a0729b-38d4`，機器 swop/Mac Studio）。
- 結構：`pipeline/`（Python 與 Node 腳本）、`state/`（大量 JSON 進度／計畫／回條，唯一權威來源）、`outputs/google-photos-api-recovery-runbook.md`（事件與規則記錄，逐日追加章節）。
- 該工作區**不是 Git repo**；證據同步靠 `state/control-backup-*.zip` 與 Drive 備份，不要期待 git 提交。

## 直譯器限制（實測踩過）

- pipeline 腳本需要 **Python ≥ 3.11**（`hashlib.file_digest`）與 `PIL`。非登入 shell 的 `python3` 可能解析到 `/usr/bin/python3`（3.9），此時 `sync_anime_drive.py` 會把每筆審閱項目降級成 `review` 並印出
  `Visual classification held: <sha> module 'hashlib' has no attribute 'file_digest'`，
  結果是「看起來成功但 total=0、checked=0」——比對錯誤更容易漏看的失敗模式。
- 正確啟動方式（本機 swop 已驗證）：

  ```bash
  uv run --offline --with pillow python pipeline/<script>.py ...
  ```

  本機 uv 在 `~/.local/bin/uv`；或直接用 `/opt/homebrew/bin/python3`（3.14，含 PIL）。

## 分類審閱機制

- `state/anime-classification.sqlite3` 的 `classified(sha,state,path,model)` 是模型分類（`anime` / `real` / `review`）。
- 人工審閱走 `state/anime-classification-reviews.json`（每筆需 `policy`、`source_sha256`、`source_path`、`original_state`、`state`(anime/real)、`review_method=assistant_direct_image_inspection`、非空 `reason`、`reviewed_at`）。
- 任一欄位不符時 `classifications()` **不報錯**，只把該筆留在 `review` 並印一行 `Visual classification held:`。送出前務必先乾跑確認：

  ```python
  import sys; sys.path.insert(0,'pipeline'); import sync_anime_drive as s
  print({h: s.classifications().get(h) for h in scope_hashes})
  ```

## 目的地政策

- 動漫圖片只進既有 Drive `Media/anime/picture`（folder id `13qVDfHp6mJP3mGWYx3N6lg_wQ5JEo_5d`），由 `sync_anime_drive.py` 用 rclone remote `gdrive:` + `--drive-root-folder-id` 複製，回條在 `state/anime-drive-items/<source_sha256>.json`；**不上傳 Photos**。
- 只有分類為 `real` 的靜態圖片（與影片）才走 Photos 上傳；`review`／`unknown` 一律排除。
- Drive 複製完成**不能**推論 Photos 舊件已可清理（`anime_photos_cleanup_not_inferred_from_drive_copy`），清理需各自的凍結計畫與逐項回讀。

## 驗證習慣

- 清理／上傳一律「先凍結 SHA256 範圍 → 逐項回條 → 純回讀」，未知寫入只回查不重送；永久刪除永遠關閉，只做可恢復垃圾桶。
- `pipeline/recovery_health.py` 是免連線的現況快照，可先看 worker 是否存活與各階段狀態。

## 雲端寫入的工作階段依賴（實測）

- 所有 Photos 讀寫都要一個「活著的 Photos 網頁工作階段」。本專案只認內建（Codex/ChatGPT app）瀏覽器工作階段；`~/.config/google-photos-migration/web-session.json` 的複製 cookie 幾分鐘內失效。
- 失效的明確指紋：`PhotosWeb` 建構時 `AssertionError: Photos web session requires refresh`，直接驗證會看到 `GET https://photos.google.com/` 回 **302** 到 `https://www.google.com/photos/about/`。此時 cookie 模式不可用，且失敗發生在任何 journal／progress 寫入之前（不會留下半成品）。
- 另一個 harness（例如 pi）**無法**借用那個工作階段：`native-browser-context.json` 記的 pipe 屬於特定 Codex session/turn，會過期；`/tmp/codex-browser-use/*.sock` 只服務當下那個 turn，外來的 `getInfo` 探測會 timeout。所以雲端步驟必須由持有內建瀏覽器的 session 執行，或由使用者先讓該工作階段復原。

## 可重用的範圍凍結模式（category-excluded trash 學到）

當清理需要雲端工作階段、但工作階段當下不可用時，把設計拆成兩段就不必等人：

1. `--plan`：只用離線可驗證的證據凍結範圍（分類審閱、本機來源／成品 SHA256、對最新 Drive 列表的 name 唯一性＋size＋MD5＋file ID＋sha256），並把 plan SHA256 寫進授權紀錄。
2. `--run`：寫入前才做雲端身分證明（`swbisb` 內容雜湊 → media key/dedup，必須與 `fDcn4b` 回讀的 dedup、檔名、位元組一致且無垃圾桶時間戳），先寫 journal `sending`，再送 `XwAOJf`，之後用 `zy0IHe` 垃圾桶回讀對帳；不確定寫入只回查不重送。執行器可重入。

這樣「缺工作階段」只阻塞最後一步，不阻塞範圍、授權、測試與文件。
