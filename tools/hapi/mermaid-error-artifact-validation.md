---
title: "Mermaid fallback 測試須觀察實際錯誤產物"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 2ea3d7d1f2582b09f2aeffa741edb8c6c5fc9bd6fefce588ebfe0d9c1dd50f85
---

# Mermaid fallback 測試須觀察實際錯誤產物

歷史助手先提供錯誤語法來測原始 code fallback，後承認這沒有測到 issue 的可見 failure，改為比較 Mermaid 自身留下的 Syntax error in text SVG 與 suppressErrorRendering 行為。

驗證渲染修補時直接觀察該渲染器的錯誤產物，區分原始 code fallback 是否可用與錯誤 SVG 是否殘留。聊天說明本身可能含錯誤文字，不能把整頁出現該字串當成唯一判據。來源自述直接比較有差異，但未提供本次重跑或部署證據；不補寫原文未出現的 selector。
