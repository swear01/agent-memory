---
title: Session sidebar 的動態置頂與捲動錨點
scope: tools/hapi
status: verified
updated: 2026-09-05
---

SessionList 依置頂、連線狀態及 updatedAt 重排。路由的 resetScroll=false 只處理導航，不能保存背景更新前的閱讀位置。

瀏覽器回歸驗證必須包含 native overflow-anchor 開啟與停用兩種模式；Chromium 原生補償會掩蓋部分缺陷。以真實 SessionList fixture 測量可見列的 bounding box，並等待收合動畫完成；jsdom 的 scrollTop 斷言不能驗證版面位移。

React 的 getSnapshotBeforeUpdate 可在 DOM mutation 前讀取可見列及專案區塊位置，componentDidUpdate 再補償。跨區塊搬移的列會 remount，應略過已移除節點，使用存活鄰近項目。

不能無條件選第一個存活錨點：該專案本身取消置頂後可能被排到最底部，跟隨它會造成大幅跳動。應比較內容座標的位移（viewport 相對位置加 scrollTop），優先保留位移最小的可見存活項目；只比较 viewport 座標會被瀏覽器原生補償誤導。

已用 Chromium 驗證十個案例：進出 Running／Active／全域置頂、可見專案移出、取消專案置頂重排、自動收合、全部收合時的專案標題，以及使用者捲回頂端。上游追蹤：tiann/hapi issue #1775。
