---
title: awk -v 會解碼反斜線跳脫，比對原始字串要用 ENVIRON
scope: tools/shell
status: active
updated: 2026-11-18
tags: [awk, findmnt, escaping, mount]
---

# `awk -v` 會解碼反斜線跳脫

`awk -v m="$x"` 會處理 `$x` 裡的 `\n`、`\t`、`\x20` 等跳脫，awk 拿到的不是原字串。`findmnt -r` 會把掛載路徑裡的空格輸出成 `\x20`，所以用 `-v m='/a\x20b'` 再跟 `$1`（仍是 `/a\x20b`）比對，永遠不相等。2026-11-18 dvlab-dashboard 的每人用量掃描就因此把要掃的掛載點自己也 prune 掉，整顆磁碟沒被掃到。

做法：用環境變數傳，awk 讀 `ENVIRON` 不做跳脫處理：

```bash
m="$mnt" awk '$1 != ENVIRON["m"]' <<<"$MOUNTS"
```

同一個腳本的其他相關經驗：

- `findmnt -r` 的 `\x20` 要在 bash 用 `printf '%b'` 解碼；dash 的 `printf %b` 不解 `\xNN`，所以腳本要用 `#!/bin/bash`。
- 同一個裝置掛在多個路徑（`/` 和 `/mnt/md0`），要依 `MAJ:MIN` 去重。`find -xdev` 擋不住同一檔案系統的 bind mount（`st_dev` 相同），必須把該裝置的其他掛載點 `-prune` 掉，否則檔案會被算兩次。
- 部署時不要直接覆寫正在跑的 sh 腳本：shell 是邊讀邊執行，執行到一半會語法錯誤。先寫 `.new`，再 `mv` 換上去，執行中的程序會繼續讀舊的 inode。
