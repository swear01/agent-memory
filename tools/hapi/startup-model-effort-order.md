---
title: "序列化 runtime mutation 不等於保證 model 先設定"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 719337174dd83c4e1948c97fb8d018193e3e8bc08d5e11dd1fc8120da380c7ea
---

# 序列化 runtime mutation 不等於保證 model 先設定

助手指出 startup 的 set_model 與 set_thinking_level 都經 promise-chain mutex，但仍認可需要加強 model-first 次序。

如果 effort 的合法值依賴 model，startup 應明確先完成 model 設定再處理 effort；互斥只處理重疊執行，還要核對呼叫入列順序與失敗處理。

來源是審查判讀及修改意圖，沒有修後結果。不推論所有 mutex 都不保序，也不併入無關的 import 控制項。
