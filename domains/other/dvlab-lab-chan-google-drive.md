---
title: DVLab Google Drive「Lab 產」考古題歸檔與 gws 操作方式
scope: domains/other
status: active
created: 2026-11-22
updated: 2026-11-22
tags: [dvlab, google-drive, gws, lab-archive, latex]
---

# DVLab Google Drive「Lab 產」考古題歸檔與 gws 操作方式

## Google Drive 位置與結構

- DVLab 實驗室考古題與修課資料位於 Google Drive「與我共用」資料夾 **`Lab 產`**（owner：`dvlabdvlab`，不會出現在本機掛載的「我的雲端硬碟」目錄下）：
  - `Lab 產` folder ID：`1PLV3CNvSL_b6YMaM7-bJV3v-91VdLDG5`
  - 內含課程資料夾，如：
    - `邏輯合成與驗證`：`1Z8vuK4Bzd1mAwdt2rojCxqJr9cyTYTwq`
      - `hw/`：`1DDXCWdOEJZ9UFvEof3zML4gwO0ESBgbF`（依民國學年分層，例如 `110-1`、`112-1`）
      - `midterm/`：`1PG1G-u74ym7T0ji7Aq3i9jHUUcYQjPl5`（含 `LSV_2026_Midterm_115-1.pdf` 與 `.tex`）
    - `CVSD`：`1aJMzmieU2XHcjfsozG_pHtAZV4q23our`（分 `hw/` 與 `midterm/`）
- 使用者具備 `canAddChildren` 與 `canEdit` 權限，可直接建立子資料夾與上傳。

## gws CLI 操作模式

本機已安裝並登入 `gws`（keyring backend），查詢與上傳共用資料夾時一律帶 `supportsAllDrives: true` 與 `includeItemsFromAllDrives: true`：

```bash
# 列出資料夾內容
gws drive files list --params '{"q": "'"'<FOLDER_ID>'"' in parents and trashed = false", "fields": "files(id, name, mimeType)", "includeItemsFromAllDrives": true, "supportsAllDrives": true}'

# 上傳檔案至指定資料夾
gws drive files create \
  --params '{"supportsAllDrives": true, "fields": "id, name, webViewLink"}' \
  --json '{"name": "<FILENAME>", "parents": ["<FOLDER_ID>"]}' \
  --upload "<LOCAL_FILE>"
```

## 本機 TeX Live 2026 Basic 編譯注意

- 本機安裝的是 TeX Live 2026 Basic（`/usr/local/texlive/2026basic`），缺少 `enumitem`、`algorithm`、`algpseudocode`。
- 撰寫可攜 LaTeX 考卷或文件時，改用內建的 `enumerate`、`booktabs`、`tikz`、`fancyhdr`、`xcolor`、`hyperref` 與 `\fbox` 偽代碼區塊，即可直接以 `pdflatex` 編譯通過。
