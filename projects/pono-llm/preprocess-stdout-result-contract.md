---
title: "Preprocess stdout 必須維持單一路徑契約"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 59d82303d75418da685c71f27dce8ec99f381d6eb6dde9e99e095f1c5e6e0f4f
---

# Preprocess stdout 必須維持單一路徑契約

使用者的 shell 管線把 preprocess stdout 當 constrained file；模型路由與 invariant 診斷一起流入 stdout，後續程式把整段當檔名而退出。只修 llm log 後仍見 preprocess 訊息污染。

沿所有被呼叫 helper 將診斷導向 stderr，stdout 只輸出約定的結果，並在 consumer 核對退出狀態與檔案存在。來源有重複失敗與修正意圖，沒有最終乾淨 stdout／端到端成功；solver 拒絕候選 constraint 與檔名污染是獨立問題。
