---
title: "Reasoning 預算與截斷要從實際回應確認"
scope: tools/llm
status: active
updated: 2026-09-16
evidence_digest: 9478746381be7c6d545a7525bf8edc0f8e540fccc711d07a8352d97076cbbcfa
---

# Reasoning 預算與截斷要從實際回應確認

歷史助手把零候選歸因於 reasoning 用掉大部分輸出預算、JSON 被截斷，並回報改設定後較快；但緊接的驗證因腳本路徑不存在退出。

檢查該 provider 的實際 token usage、停止原因與原始結構化回應，將截斷或解析失敗與真正零候選分開。是否關閉 reasoning 或增加預算須依模型契約與實测，不能當成通用修法。來源中的 commit 訊息與延遲自述不等於可靠注入已通過。
