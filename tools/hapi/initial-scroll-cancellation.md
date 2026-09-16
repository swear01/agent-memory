---
title: "初始化置底不能把所有 scroll 都當使用者取消"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: b3dba129b5a8ac83634137cf5e69c8450626777b25de60db0376d1e20467b96e
---

# 初始化置底不能把所有 scroll 都當使用者取消

歷史助手回報切入長 session 後已到頁底，隨即一次向上 scroll 使畫面回到頂部，初始化保護期後仍未恢复；當時將問題縮到 HappyThread 的取消計時器與關閉 auto-scroll 路徑。

初始化期間區分使用者捲動意圖與瀏覽器恢復、DOM 調整等程式造成的 scroll，避免非使用者事件永久取消後續置底。來源只到重現與定位，未改程式，也未查明是哪個更新引入；不把 sidebar 列表錨點問題當成同一機制。
