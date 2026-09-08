---
title: "遠端分支已不存在時，清理 stale tracking ref"
scope: "tools/git"
status: active
updated: 2026-09-08
evidence_digest: e1fb5d001811f7bc32ebea4b60c1e19062c4ed306fa0a962ddfaed2a2459c141
---

# 遠端分支已不存在時，清理 stale tracking ref

## Problem

歷史批次刪除分支時，一個名稱回報 remote ref does not exist，另一個名稱刪除成功；後續 prune 清掉已過時的 origin 追蹤參照。

## Durable rule

將每個分支的結果分開核對。遠端已不存在的分支不需要再刪一次；清理本機 stale remote-tracking ref，避免把舊清單當成遠端現況。

## Boundary

此記錄只證明一次刪除失敗與追蹤參照清理；prune 不是讓不存在的遠端分支重新變得可刪除，也不能把整批非零退出碼當成所有分支都沒刪掉。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
