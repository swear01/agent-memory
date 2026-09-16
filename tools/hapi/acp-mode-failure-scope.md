---
title: "無效 mode 不代表整個 set_mode 能力不存在"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: ffa7d335f6a6cea341ebe29e22c31057a2c7c354ca3953ac29c10e6cf9e82431
---

# 無效 mode 不代表整個 set_mode 能力不存在

助手接受 Copilot ACP review：Interactive 被做成 no-op，只更新 HAPI 狀態而 backend 留在舊 mode；另一個分支把 Invalid mode 當能力不存在，永久擋掉後續合法模式。

依實際公布的 mode 做映射與確認，只有 backend 成功切換才同步成功狀態。區分單個值被拒絕與方法不支援，不能用 no-op 偽裝使用者要求已生效。

當時 Interactive 對應 agent 是該整合的歷史修法，不能套到所有 provider。來源沒有完整修後測試；不採用較早的 no-op 方案。
