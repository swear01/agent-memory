---
title: "PDF 產生不代表要求的字型特性已生效"
scope: tools/latex
status: active
updated: 2026-09-16
evidence_digest: cdfc3334e923a7c9308ee9f3834ae541d764ff6dd2a691f6ff7e83192c38ab5f
---

# PDF 產生不代表要求的字型特性已生效

來源 build 明確警告所用 Noto 字型缺少 Numbers=Monospaced 的 tnum 特性，並有字型形狀替代，最後仍輸出 PDF；助手只提沒有 overfull。

分別檢查建置結果、字型替代與指定 OpenType 特性是否支援。沒有 overfull 不會消除其他 warning，也不能證明數字等寬或視覺效果符合要求。來源沒有檢視後的確認或修正結果，Medium 字重缺失與 tnum 特性分開處理。
