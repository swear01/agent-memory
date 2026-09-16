---
title: "建立 release draft 不能跳過约定的遠端驗證階段"
scope: tools/github
status: active
updated: 2026-09-16
evidence_digest: b9ff51c05b21fc1d27661a7f0ba60b5ddada7ac671e017d07996e2e1968f8203
---

# 建立 release draft 不能跳過约定的遠端驗證階段

歷史獨立審核摘要拒絕發布流程，原因是腳本建立 draft 後立即無條件公開，少了該案要求的遠端 draft 檢查、prerelease 標記與 sealed manifest 重驗。

若發布契約要求遠端產物驗證，將驗證結果作為公開前的實際條件；其他 build 或測試綠燈不能代替。Prerelease 與封存規格依當次契約，不是所有 release 的通則。來源只有 REJECT 與其餘通過項目的摘要，沒有修正後重審或發布成功。
