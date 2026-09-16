---
title: "網路 BFS 的上限要計算契約中的網路方塊"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: f00b8908baab9315cfa8b1f56d60613d924fa805762a7585eb9bc3192c22b271
---

# 網路 BFS 的上限要計算契約中的網路方塊

使用者指出大型合法網路的 BFS 把空氣也加入受限 visited 集合，導致尚未遍歷到 Core 就先耗盡 MAX_NETWORK_BLOCKS。

分清防止重訪的搜尋狀態與受業務上限計數的網路節點，對三個 traversal 保持一致契約。驗證已載入網路方塊與周圍空氣很多的合法網路仍可找到 Core。

來源是已定位缺陷的修復指示，沒有後續 RED/GREEN 輸出；當時只准修改單一檔案的限制不升格成通用規則。
