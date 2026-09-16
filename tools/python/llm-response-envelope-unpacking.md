---
title: "LLM 回傳封裝須先解包再解析文字"
scope: tools/python
status: active
updated: 2026-09-16
evidence_digest: e7a78da485df933c8515519392ed9f2b0300ce2b57aea7f5c54d7d77798618ae
---

# LLM 回傳封裝須先解包再解析文字

歷史原始輸出顯示 LLM 呼叫回傳的是含 JSON 文字與統計資料的 tuple。呼叫端把整個 tuple 傳入 json.loads，得到「not tuple」的 TypeError；例外處理又把同一 tuple 傳給 re.search，再次因型別錯誤退出。這是本地呼叫契約錯誤，不能據此判斷模型沒有產生候選。

在 provider 封裝與解析器之間依實際回傳契約解包，將文字和統計資料分開，再做 JSON 與候選 schema 驗證。輸入型別錯誤應明確回報，不能用 regex fallback 重複處理同一個非文字值。

來源包含原始 tuple、兩層 traceback 與助手承認需要解包的回覆；未提供修正後整條流程通過的證據，也不證明 tuple 裡的候選不變式正確。
