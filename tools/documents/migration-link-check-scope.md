---
title: "文件搬移後要查實際連結並區分既有與新斷鏈"
scope: tools/documents
status: active
updated: 2026-09-16
evidence_digest: aa52903c2164c0aac6c1b503d3f637f064b00f58fbd3999dda07c2d4a50266f0
---

# 文件搬移後要查實際連結並區分既有與新斷鏈

歷史助手先由 active 文件少有 archive 引用推論遷移沒造成斷鏈，後續 checker 卻列出 22 個 broken links，並承認 overview 指向 schema 的連結可能是自己新增。

在受影響文件實際解析相對路徑、確認目標並與搬移前比對，分開記既有缺檔、新引入及 checker 限制。沒有引用 archive 不能證明所有連結乾淨；來源只到重新調查，沒有最終零斷鏈報告。
