---
title: Google Drive 實驗室座位表與本機列印
scope: tools/documents
status: active
created: 2026-09-29
updated: 2026-09-29
---

# Google Drive 實驗室座位表與本機列印

## Durable rule

- Grok Bot / Cursor 的 Google Drive 連接器 id：`user-Google-drive`（需授權後才有工具）。
- DVLab 115 座位表檔名：`DV_Lab_115_座位表`（有 `.pdf` / `.png` / `.pptx` 三版，同一 Drive 資料夾）。
  - PDF file id：`1ByCnoHK8mH89L863D_qsau2FAf3P4uzV`
- 需要本機印表機時：用 connector / `download_file` 拉到 box，再 `CopyFromBox` 到目標 Mac 的 `~/Downloads/`，或直接用本機已同步的 Google Drive 路徑。
- 列印細節見 `machines/swairM5/kyocera-ecosys-m6635cidn-ipp-print.md`。

## Related local path（曾用）

- GUPS 工讀簽到：`…/GoogleDrive-stanley.yellow1@gmail.com/我的雲端硬碟/document/工作紀錄/碩一/GUPS助教/附件3_工讀生簽到退表_115年9月_已簽名.docx`
