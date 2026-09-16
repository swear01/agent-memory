---
title: "極端 recipe GUI 測試不能以材料聚合代替位置圖"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 6de558c6059600ab49b2f7d40491116c349319bc9389364642311e558283e9f4
---

# 極端 recipe GUI 測試不能以材料聚合代替位置圖

歷史唯讀審查指出 81 個位置被壓成最多九組材料塞入 3×3 presentation；交易消耗量正確仍沒有測到真正 9×9 版面，rollback fixture 又可能先撞 type cap 或消耗後釋放 slot。

分開測位置圖、材料數量與輸出容量拒收，讓 fixture 在消耗後仍維持預期容量條件。來源只跑 static checks，未執行 GameTest／client GUI，也混有 resize、scroll、搜尋與 chemical 等独立 findings；本筆記不合併其原因或宣稱任何 runtime 修復。
