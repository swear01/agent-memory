---
title: "忽略 skill 內容不等於排除整個 skill"
scope: tools/skillshare
status: active
updated: 2026-09-16
evidence_digest: 744f33b9eb2953770be1804d206aca83743699fb80d22f5b44dcead1d682d912
---

# 忽略 skill 內容不等於排除整個 skill

歷史助手診斷 shelf skill 仍出現在多個 target：當時 config.yaml 的 ignore 處理 skill 內部檔案，沒有將整個 skill 排除於 discovery，因此提出改用來源根目錄的 .skillignore。

排除設定先辨識作用層級，驗證實際 discovery 清單與既有連結，不能只確認設定檔有寫名字。來源只有診斷與修正計畫，沒有清除連結後結果；使用前依安裝版本重新確認 ignore 語意。
