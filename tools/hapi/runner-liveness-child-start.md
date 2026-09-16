---
title: "Runner 在線與子程序啟動是不同驗證層"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 9539e924988924d2308a479d36daa8c552f8aeafedbf6ea1ce0799f9d6987cd3
---

# Runner 在線與子程序啟動是不同驗證層

歷史診斷中 runner heartbeat、PID、控制埠和 hub 回應正常，但新建 Codex session 的 child 立即以 code 1 結束，只留下籠統錯誤。

先沿失敗 child 的實際啟動鏈收集 stderr、工作目錄與權限證據，分開報告 runner 存活和 session 可用性。不能因 hub 回 200 就稱可正常工作，也不能未查原因便假設重啟可解。來源沒有最終 child 失敗根因或修復結果。
