---
title: "本地 Claude spawn 要隔離 SDK entrypoint 污染"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 6f7ad8cebf58620ec019b8f34b874baf6d33773664a5a8c72d03d0c705363b11
---

# 本地 Claude spawn 要隔離 SDK entrypoint 污染

來源程式註解指出 SDK metadata extraction 會在父程序設定 CLAUDE_CODE_ENTRYPOINT，洩漏到本地 child 後可能讓 session 被當成 SDK 啟動而不列於 resume；程式已從 inherited env 移除此欄位。

在 spawn 邊界區分 parent metadata 與 child 模式，清除非本地契約的 SDK 標記並保留必要的使用者設定。來源仍在 cleanEnv 後套用顯式 claudeEnvVars，故不可聲稱所有途徑都禁止該值；此處只有程式片段，沒有實際 resume 成功或部署證據。
