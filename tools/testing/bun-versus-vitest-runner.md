---
title: "vi.hoisted 失敗先核對實際測試 runner"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: fdd4e6e6d7364cedecb1182b0234de2c5f2634fd21ff2fb0e7fc79dd107e1d97
---

# vi.hoisted 失敗先核對實際測試 runner

原始輸出明示 bun test 執行引用 Vitest 的 launcher tests，載入時 vi.hoisted 不存在。另一次 Vitest 執行則失敗於 workspace alias 解析，兩者不是同一原因。

從命令、runner banner 與專案測試設定確認真正的執行器，再處理 API 或依賴。不能看見 Vitest import 就斷言這次用的是不相容 Vitest 版本。

保存輸出只有失敗與部分測試通過，沒有整套修復成功；不把所有 Export not found 或套件解析錯誤都歸咎 runner。
