---
title: "插入目前模型時不要切斷既有分組"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 0985ee254fd3549de400303770e815668b3077396fbc34053c9294eb4fe40fed
---

# 插入目前模型時不要切斷既有分組

使用者指出模型選單出現太多分類；歷史助手定位到 mergeModelOptions 把 Current 插在 Native 區塊中間，而渲染器每逢 group 改變就再畫標題。

合併缺少於新 catalog 的目前選项時維持群組連續，並測目前模型仍保留但不重複分組的畫面。這不是多個 Native catalog 的證據，也不能因清單更新就擅自替 session 換模型。來源只有診斷與放置方案。
