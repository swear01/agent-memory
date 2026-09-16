---
title: "模型設定與測試預期不同時先確認哪個是契約"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 79b589a30a4750d9fadb247f31acec28e829eb1f2d704efc78e6ea8d499ba9b4
---

# 模型設定與測試預期不同時先確認哪個是契約

原始 pytest 顯示 config 載入的是 preview model，測試仍期待另一名稱；同一來源有未提交 YAML diff 將名稱改為 preview，助手便說測試斷言過時。

先確認當次授權的預設值與設定變更，再同步預期，不能只把測試改成跟實作一樣就宣稱正確。保留使用者未提交設定，核對載入路徑及環境覆寫。來源没有修改後測試通過；註解所述免費額度與 thinking 行為未驗證，不作現行模型建議。
