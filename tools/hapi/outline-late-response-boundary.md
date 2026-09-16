---
title: "共用載入結果套用時重查原呼叫意圖"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 83e7f3aee3e538809ad34d85bb62eca56572a3f1030d9057268f7270592fecd4
---

# 共用載入結果套用時重查原呼叫意圖

歷史 review 指出 outline 關閉後，in-flight consumer response 仍可能安裝 outline boundary；同一路徑又被不需 outline 的 tool-group hydration 使用。

把資料取得與呼叫者專屬副作用分開，在套用當下重查 outline 是否仍啟用，以及此次請求是否屬於該用途。取消或版本標記應符合既有載入生命週期，不能讓晚到回應恢復已關閉狀態。來源是 finding 與獨立 outline source 的提案，未證明已修復。
