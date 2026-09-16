---
title: "Hook 測試的參考穩定性須符合真實 app"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 651941b4f8a3e5e7e52b0dfca2f611b44fcaad1d6152bf1815f659a9c187bcda
---

# Hook 測試的參考穩定性須符合真實 app

歷史助手初版 hook test 每 render 傳新的 authSource 物件，製造 update-depth 迴圈；查到 app 使用穩定 state 參考後修 fixture，才隔離 token refresh 重建 ApiClient 的問題。

保留真實依賴生命週期，先排除 fixture 引入的額外觸發，再用指定失敗斷言驗證修法。來源回報 memo 依賴改為 token 存在性後 RED→GREEN 與全 suite 通過，但 minified stack overflow 尚未因果確認，也没有部署結果；不要把所有畫面故障合成同一已修根因。
