---
title: "單一 workflow 沒警告不代表其他 Actions runtime 已更新"
scope: tools/github
status: active
updated: 2026-09-16
evidence_digest: e20edc74f5ab870ca6edb93260213ba0374fe771b265ddeceded7839aad86686
---

# 單一 workflow 沒警告不代表其他 Actions runtime 已更新

歷史助手升級 setup-python 後回報該 run 無 Node 20 annotation，後續清單仍找到 release workflow 的 cache、artifact 及 release action 使用 Node 20。

依每個 workflow 實際引用的 action 與其 runtime 盤點，先核對 major breaking changes 和 runner 要求，再驗證受影響 workflow。目標 Python 版本不是 action 自身的 Node runtime；來源的成功限已執行那條 workflow，不是全部 release 路徑已更新，版本數字也不是今日建議。
