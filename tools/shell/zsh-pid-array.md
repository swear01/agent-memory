---
title: "多個 PID 不要依賴 zsh 字串自動分詞"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: 2c7dff7511479a4178ffb071432ee57b58c01f31df30d357b1fe0d8bc0e685af
---

# 多個 PID 不要依賴 zsh 字串自動分詞

助手自述把兩個 PID 放在一個含空格字串中傳給終止命令，zsh 沒有自動拆成兩個參數，舊 client 因而仍開著測試世界；後續出現存檔錯誤，該次實驗被中止。

使用陣列或逐項明確傳遞已確認歸屬的 PID，並檢查程序確實退出，再開啟同一測試資源。shell 命令已送出不是清理完成證據。來源說已改用 PID array，沒有重新生成世界後的測試結果；不把受污染結果當有效實驗。
