---
title: Gmail connector 沒有附件下載工具時用 RAW 取出附件
scope: tools/mcp
tool: Gmail MCP connector
status: verified
created: 2026-10-19
updated: 2026-10-19
tags:
  - gmail
  - attachment
  - mcp
---

- claude.ai Gmail connector 的 `get_thread` / `get_message` 只回附件 metadata（`attachmentIds`、`filename`），沒有下載附件的工具；`get_thread` 也不支援 `RAW`。
- `get_message` 帶 `messageFormat: "RAW"` 會回傳整封 MIME 的 base64url（含附件）。小信件（數十 KB）可直接用。
- 工具結果不會自動存檔，但會寫進當前 session transcript JSONL（`~/.claude/projects/<cwd 編碼>/<session-id>.jsonl`）。用 Python 找含 `"raw":"` 的 `tool_result`，`base64.urlsafe_b64decode` 後交給 `email.message_from_bytes`，對有 `get_filename()` 的 part 取 `get_payload(decode=True)` 存檔。
- 同樣做法適用內建瀏覽器 `javascript_tool` 回傳 base64：結果超過上限時會存到 session 的 `tool-results/*.txt`（JSON 陣列），從那裡解碼即可。
- 存檔後用 `pdftotext` 核對發票號碼等內容再放進目標資料夾。
