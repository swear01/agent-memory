---
title: HAPI release review timeout and provider evidence
scope: tools/hapi
status: active
updated: 2026-09-06
---

# Review evidence

- Swear Review 的自動 push review 是整份 PR 的 full review。先比較 base 與 head 的 numstat，將 maintained-source replay patches、生成資料與實際程式差異拆開，避免把補丁快照行數當成新寫程式碼。
- 已驗證 OCR runner 透過 hard_timeout_minutes 計時，到期設定 timedOut 並 SIGKILL 自己的 process group；worker 報成 cancelled / timed out。這不同於新 push 導致 superseded (cancelled)，也不證明服務宕機、模型額度耗盡或有人手動取消。
- 服務及 gateway active 只證明程序存活。若沒有模型輸出或請求日誌，不能宣稱已找到模型停滯根因；不要對相同 head 反覆等待同一個硬逾時而沒有新診斷。
- provider 的舊 quota 拒絕只能代表該次 review。Gemini GitHub review 額度與 CLI 分開；先確認 Developer Connect repository link 及安裝 COMPLETE，再依官方 reset 時間判斷是否值得重試。eyes reaction 是接收證據，不是 clean review。
- 外部 review 必須對應目前 head；有 findings 的 COMMENTED 不能當通過。先核對 helper 與回歸測試，例如 withSettingsFileLock 使用 realpath: false 與 sidecar lock，不要求 target directory 已存在，不能未查實作就採用先建立目錄的建議。
