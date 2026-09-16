---
title: "產生 PDF 檔不等於內容與字型有效"
scope: tools/latex
status: active
updated: 2026-09-16
evidence_digest: 38c51862a584964e3d36722216f65aeab5a2fd0abd3d354d971b3d2c74243deb
---

# 產生 PDF 檔不等於內容與字型有效

使用者 log 顯示 lualatex 產生三頁小檔卻伴隨 nullfont，pdffonts 無字型且抽文字只有換頁；另一輪更直接 fatal。改用 tectonic 後有較大 PDF 與嵌入字型，但列出的 family 是 JP。

以實際頁面、文字與字型檢查成品，檔案存在或大小只能是線索。缺 luaotfload 工具／cache 不足以排除所有其他根因，切換引擎成功寫檔也不能證明 TC 字形、版面或可重現性；來源沒有最後視覺驗收。
