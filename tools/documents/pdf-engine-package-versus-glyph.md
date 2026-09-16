---
title: "PDF 引擎套件不足與字型缺字要分階段驗證"
scope: tools/documents
status: active
updated: 2026-09-16
evidence_digest: 65cc6757c76728719a88720b5397ab76ea6baaea52ebfa1db163fb99d9da8935
---

# PDF 引擎套件不足與字型缺字要分階段驗證

原始 Pandoc log 先因缺 lualatex-math.sty 停止；改引擎後產生 21 頁 PDF，仍有近似與小於等於符號缺字及版面警告。

分別確認引擎依賴、字型覆蓋與實際頁面渲染。成功產檔或引擎處理 Unicode 不保證所選字型有字形；替換字型後仍需讀回受影響符號與版面。來源只到準備修字型，沒有最終無缺字或目視通過證據。
