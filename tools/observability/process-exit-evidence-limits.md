---
title: "程序消失調查要分開直接證據與無法排除項"
scope: tools/observability
status: active
updated: 2026-09-16
evidence_digest: be54ee6eca6e9ad76b6671ba434b8b722b222eea810ba504650a1715a4294bf4
---

# 程序消失調查要分開直接證據與無法排除項

使用者回報找到另一 session 送出 signal 的直接證據，要求停止廣泛搜尋；歷史助手列出 boot 資料、可讀 cgroup OOM 計數與無權讀取的 kernel log。

先處理已知直接線索，再明確標註各項反證的覆蓋範圍。特定 cgroup 沒有 OOM 計數不能排除所有全域機制；service result 為 resources 也不能直接當主機資源耗盡。來源的 signal 結論屬使用者與助手回報，本次未獨立重建事故。
