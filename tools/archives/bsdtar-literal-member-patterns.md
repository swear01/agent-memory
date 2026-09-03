---
title: bsdtar 精確選取含 glob 字元的 archive member
scope: global
tool: bsdtar
tags: [bsdtar, libarchive, archive, glob, rar]
status: active
created: 2026-09-04
updated: 2026-09-04
---

# bsdtar 精確選取含 glob 字元的 archive member

`bsdtar -x` 與 `bsdtar -t` 的 positional operands 是 shell-style glob patterns；即使透過 Python `subprocess`、沒有 shell 介入，`[abc]`、`*`、`?` 仍由 `bsdtar` 解讀。`--` 只結束 option parsing，不會關閉 pattern matching，shell quoting 也無法改變這點。

因此 archive 內若有 literal 名稱 `[SumiSora][08].mkv`，這個命令可能回報 `Not found in archive`，但不代表 archive 或 payload 損壞：

```text
bsdtar -xOf <archive> -- '[SumiSora][08].mkv'
```

先以不帶 selector 的 `bsdtar -tf <archive>` 核對 raw name 與唯一 occurrence，再把 `\`、`*`、`?`、`[`、`]` 各自加上反斜線，傳給 `bsdtar`：

```python
def bsdtar_literal_pattern(path):
    return "".join("\\" + char if char in "\\*?[]" else char for char in path)
```

例如 selector 應傳成 `\[SumiSora\]\[08\].mkv`。修正後仍須檢查 return code、stderr、實際輸出 bytes 是否等於 declared member size；需要內容身分時再計算 SHA-256。這能區分 scanner selector bug、重名／正規化問題與真正 payload 問題。
