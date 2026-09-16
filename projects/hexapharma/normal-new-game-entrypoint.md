---
title: "開發用 seed query 不能代替玩家的新遊戲入口"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: c45e10bc67c0c127980c36458755a3a75a03d30006542fe3fd6ddf208a5c4f1a
---

# 開發用 seed query 不能代替玩家的新遊戲入口

歷史 read-only audit 指出 seed 只來自開發 query、預設固定值，正常 HUD 沒有建立另一 seed 的動作；這使跨存檔重玩目標只能靠開發網址操作。

驗證玩家從正常入口建立、保存、載入不同遊戲的完整流程，不以引擎支援多 seed 就當產品可達。來源另列 touch、經濟顯示、modal 與 HUD 問題，機制分開保留；實際 Playwright 證據限 viewport 幾何，沒有修後新局流程通過。
