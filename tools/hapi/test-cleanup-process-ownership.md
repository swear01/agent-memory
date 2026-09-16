---
title: "測試 child registry 不應接管 runner 的關閉責任"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 46d85045d704fa4cb0286e2e709688e32309125fcedd6723e6b4a2427b13fa08
---

# 測試 child registry 不應接管 runner 的關閉責任

歷史助手回報新增的 stage-2 tree-kill 在 afterEach 殺掉 runner，跳過原本 stopRunner 的 graceful cleanup；留下 stale state，下一個測試又把死 PID 當成已啟動。

將 runner 與 session／terminal child 的清理責任分開，讀回 state 時核對實際程序身分與可用狀態。測試 registry 的 broad kill 不能代替 supervisor 的關閉流程。來源只有診斷與修正方向，另五個失敗仍在分類，不能全當 flaky 或稱已修好。
