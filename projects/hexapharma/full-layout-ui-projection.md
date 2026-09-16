---
title: "多列 layout 不可在 UI adapter 截成第一列"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 1aa5093f665829c95599403510a434b5cd1658c63018341d6d9898637ca5e259
---

# 多列 layout 不可在 UI adapter 截成第一列

使用者指出 compileTemplate 已輸出多列、含 L 形或二乘二機器的 layout，但 Factory recipeLayout 仍只複製 row zero；助手也確認既有 adapter 的截斷。

讓 UI 使用完整 width、height、tiles 與 machines，並以非首列有有效機器的案例驗證顯示和執行。Compiler 正確不代表 consumer 沒有丟資料。

來源沒有修後完整遊戲結果。新地圖數量需直接驗 map count，seed 改變只證明重新生成，不能代替三張地圖的斷言。
