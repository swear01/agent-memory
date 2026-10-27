---
title: HAPI Markdown 檔案預覽的相對連結／圖片（#1917）與 session 檔案讀取沒有大小上限
scope: projects/hapi-markdown-preview-relative-assets
project: hapi
status: active
created: 2026-10-27
updated: 2026-10-27
tags:
  - hapi
  - web
  - markdown
---

# 問題與修法

- 根因：`web/src/routes/sessions/file.tsx` 的 standalone `MarkdownRenderer` 只拿到內容、不知道檔案路徑；沒有 chat context 時，相對 `.md` 連結落入 fail-closed policy 變成 inert，相對 `<img src>` 則對 web 頁面 URL 解析。
- 修法（PR tiann/hapi#2211，HAPI Bot 在 head `9ac489e` 通過，等待 upstream merge）：`MarkdownFileBaseContext`（sessionId、filePath、workspacePath、fileSearch）＋ `resolveMarkdownRelativePath()`（`markdown-href-policy.ts`，重用 `resolveLexicalPath`／`isWithinWorkspace`）。連結重用 `FilePathAnchor`，圖片走 `api.readSessionFile` 轉成 `data:` URL。

# 值得記住的坑（bot review 抓到的）

- CLI 的 `readFile` RPC handler **沒有大小上限**，回傳 base64 會撞 hub socket 約 48 MiB 上限。任何「自動」讀檔（例如預覽時載入圖片）都要先限制：web 端沒有 stat API（`statFiles` 只在 hub 的 git route 內部用），可以用既有的 `listSessionDirectory`（entries 有 `size`）先看大小，再決定要不要讀。
- 相對路徑解析要先處理 `parseSessionPathHref`（`./sessions/<id>` 是合法的 session 引用寫法），再當作檔案路徑。
- `FilePathAnchor` 原本把 `origin=chat` 寫死；`useAppGoBack` 靠 `origin` 決定回 chat 還是 Files。從預覽跳到其他檔案的連結要沿用目前的 `origin`／`tab`／`query`。
- 預覽的 filePath 可能是絕對路徑（從 chat 連結進來），所以用 `..` 跳出時，光靠 lexical root 檢查擋不住，要再用 session `metadata.path` 做 workspace containment。
