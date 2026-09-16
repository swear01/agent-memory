---
title: "清理 generation work 前先保存可重播 stimulus"
scope: projects/ICCAD2026B
status: active
updated: 2026-09-16
evidence_digest: bcd20b0ad56bc3b7ec634bc2731c30ba2385d5eb7cc65dc255d6e23ed1b8afcf
---

# 清理 generation work 前先保存可重播 stimulus

使用者指出 backend 會刪 seed evidence；助手確認 seed.yaml、assembly 與 binary 隨 work 清掉，舊 logs 雖能支持曾跑 mutant simulation，仍缺完全相同 stimulus 的 clean 對照。

打包時先保存 generator／RTL seeds、實際 binary 與版本／hash／verdict，驗證可讀與配對後才清理工作目錄。seed 不等於 exact binary，既有 artifact 也不能稱這個 session 親自執行。來源修正了規劃範圍，但未改 backend 或啟動新 VCS，不能將預計 quota 當已完成 dataset。
