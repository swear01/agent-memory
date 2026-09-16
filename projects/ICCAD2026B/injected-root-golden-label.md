---
title: "歷史 bug 的 golden label 以注入 root patch 為準"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: 99b2f7bb8b504b980fd5afe849b48e512d773d4e74fb741b2fd509dcfd0cbc0a
---

# 歷史 bug 的 golden label 以注入 root patch 為準

歷史助手釐清 retry pool 只是完整資料的一部分，同一 root 在不同 test／seed 下卻仍帶 test suffix，讓一個原因被拆成不同 golden buckets。

保留 fix provenance 與精確 root change，以注入 patch 的語意身份合併 labels，不能用 commit、test 或 log 症狀代替。來源報出的 verification pool 尚未完成同 stimulus clean PASS／mutant FAIL、無 verdict 排除、去重與洩漏審查；數量與一次 failure 不代表正式 benchmark 已合格。
