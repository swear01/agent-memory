---
title: "模型過濾 hook 必須逐個消費入口驗證"
scope: tools/pi
status: active
updated: 2026-09-16
evidence_digest: 663686d55a3b25f4f1134d9c78b7fe853d2c7b81b963ae063d272eb611605a23
---

# 模型過濾 hook 必須逐個消費入口驗證

助手回報該版 Pi 的 CLI list 直接讀 ModelRuntime，而既有 filter 只 patch 擴充 API 的 ModelRegistry，因此不能單靠 RPC/ACP 過濾結果宣稱 CLI 也同樣受限。

核對各入口實際讀取的 registry 與 hook 路徑，分別驗證有效模型集合。已授權的範圍之外保留其他模型和非互動行為，不以新 wrapper 的提案冒稱全路徑修好。

這是歷史版本的診斷；當時 wrapper 尚待完成，安裝語法是獨立問題，沒有本筆中的全入口驗證。
