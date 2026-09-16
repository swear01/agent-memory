---
title: "SSH 非互動搜尋要明確指定 rg 的檔案範圍"
scope: tools/shell
status: active
updated: 2026-09-16
---

# SSH 非互動搜尋要明確指定 rg 的檔案範圍

本次遠端 canonical 去重搜尋曾在 SSH 非互動輸入下省略 rg 的 path。當 stdin 被選為搜尋來源時，空結果無法證明工作目錄中沒有既有教訓；改以明確檔案路徑或 rg pattern . 搜尋，才找到已存在的同原因規則並修正候選 placement。原先 placement 與來源證據均保留。

在 SSH、pipe 或 python3 - 傳入腳本的環境，讓搜尋目標顯式可查：先確認目錄存在，再傳入已知檔案或句點。不要猜子目錄後把錯誤忽略；分辨 exit 1 的無匹配與 exit 2 的輸入錯誤，並檢查 stderr。

去重結論須建立在 fresh canonical 的完整適用範圍搜尋與全文比較上。工具成功或空 stdout 不是語意無重複證據；找出同主因規則也不代表可以合併相鄰工具輸出的其他獨立事件。
