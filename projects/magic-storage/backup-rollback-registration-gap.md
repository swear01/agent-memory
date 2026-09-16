---
title: "備份搬移與 rollback 登記之間不能留下空窗"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 5752c8a8e7f55a2ccd5b96c74425e8aef4d547ce1da32784a7191c33f9ec71e3
---

# 備份搬移與 rollback 登記之間不能留下空窗

歷史 sidecar probe 回報 backup helper 已搬走兩類 jar，但回傳值尚未賦入 rollback lists 就發生 BaseException，properties 可還原而 active jars 留在 backup 目錄。原九項測試仍通過。

把每次副作用的恢復資訊在同一責任範圍內即時登記，對 handoff、部分 replace 與下載前後設語意定位的故障注入，核對完整前後樹與原始例外。來源未修程式，hash pin 正確也不能證明 rollback 安全；Python 補償仍不覆蓋 SIGKILL／斷電。
