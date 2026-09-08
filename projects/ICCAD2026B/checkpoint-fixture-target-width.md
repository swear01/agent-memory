---
title: "Checkpoint fixture 必須對齊 RTL target 與指令寬度"
scope: "projects/ICCAD2026B"
status: active
updated: 2026-09-08
evidence_digest: 9dfa84003d07d82ecdb1c5dea70125f46cbbcba46ba94753c51fa60025769e4e
---

# Checkpoint fixture 必須對齊 RTL target 與指令寬度

## Problem

ib018 checkpoint 測試模板仍指向 rtl/ibex_decoder.sv，但 root patch 的目標是 rtl/ibex_compressed_decoder.sv；模板還把 .2byte 0x9006 寫成 .word 0x9006。

## Durable rule

將 checkpoint 測試的 RTL 路徑與實際 patch target 對照，並核對組語 directive 寬度。先修正 fixture 再解讀測試失敗，避免把模板錯誤歸因於 VCS 或被測 RTL。

## Boundary

歷史 assistant 回報 runner unit test 修正後通過、接著重跑完整 suite；片段沒有完整 suite 結果或 VCS 驗證輸出。本條不把 unit-test pass 升格成模擬器或資料集驗證通過。

## Verification

控制者核對了保留的歷史來源片段。上述現象、使用者糾正或負面結果來自該記錄；本次未重新執行當時的測試或修復。來源未證明的原因與修復成功不予採用。
