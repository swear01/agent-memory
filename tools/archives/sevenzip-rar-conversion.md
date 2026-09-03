---
title: macOS 上安全將不相容 RAR 轉成 7z
scope: global
tool: sevenzip
tags: [rar, 7z, bsdtar, macos, archive]
status: active
created: 2026-09-03
updated: 2026-09-03
---

# macOS 上安全將不相容 RAR 轉成 7z

Homebrew `sevenzip` 26.02 的 `7zz` 可能能辨識 RAR5 清單，卻在測試或解壓時對每個檔案回報 `ERROR: Unsupported Method`。同一個封存檔可能仍可由 macOS 現有的 libarchive `bsdtar` 正常讀取與解壓。

安全轉換流程：

1. 保留原 RAR，不先刪除。
2. 用 `bsdtar -xpf source.rar -C "$tmp"` 解到 `mktemp -d` 建立的暫存目錄。
3. 用 `7zz a -t7z -mx=9 -m0=lzma2 -ms=on -mmt=on output.7z input-directory` 建立 7z。
4. 執行 `7zz t output.7z`，再解開新 7z，以 `diff -qr` 對照原解壓目錄；同時核對檔案數與未壓縮位元組數。
5. 將新 7z 放到目的地並確認雲端上傳完成後，才把舊 RAR 移到垃圾桶。

不要把 `7zz t` 的零退出碼單獨視為 RAR 可完整解壓的證明；仍須檢查輸出是否含 `Unsupported Method`。新 7z 的驗證則以 `Everything is Ok` 加上重新解壓逐檔比較為準。
