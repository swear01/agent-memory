---
title: "搜尋顯示可疑時直接讀取原始識別字"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: 96568a54d908276442fd51c218f7cb77f3188e4ae9601515fcbef1e89fccace6
---

# 搜尋顯示可疑時直接讀取原始識別字

歷史助手先把 collaborationMode 誤看成 ln，準備建立平行縮寫；直接讀檔後才更正，內部其實使用完整名稱。

搜尋片段若缺字或互相矛盾，先用原始檔內容確認識別字，再推導映射或修改程式。來源把原因描述成顯示假象，但沒有確認是 grep、渲染或其他中間層故障，也沒有功能修正完成證據。
