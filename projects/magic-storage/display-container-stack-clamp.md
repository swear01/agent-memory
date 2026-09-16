---
title: "顯示數量被夾住先查 display container"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 5e6996a24a27b883b3e079d8dfbb14c06acf3aeed1be542d7215b6ee32c0f41e
---

# 顯示數量被夾住先查 display container

助手指出 SimpleContainer.setItem 會套用 maxStackSize，導致 GUI 顯示被夾在六十四；當時封包 count 路徑沒有同一個 clamp。

沿實際顯示資料的 consumer 查數量限制，只調整 display-only container 的契約，保留真正取物 click path 的數量與權限檢查。

来源含診斷與 production 修改自述，但沒有修後測試輸出。最初因缺 player 的測試失敗不是有效的功能 RED。
