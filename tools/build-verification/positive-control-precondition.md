---
title: "Positive control 必須真的符合接受條件"
scope: tools/build-verification
status: active
updated: 2026-09-16
evidence_digest: 099d261e06f63eaac3c2399965281a7135af0694536a18d47f32d9b747f6b08f
---

# Positive control 必須真的符合接受條件

歷史 guard 驗證把仍在 workspace roots 之外的路徑當 positive control，得到的仍是拒絕，未能證明 accept 分支；助手後來更正以真正位於 roots 內的路徑重試。

核對正反控制組各自的前置條件，再比較預期結果。兩次拒絕不等於邊界兩側都通過，單一錯誤分支也不能證明接受路徑正常。來源自述後來取得外部403與內部200，但本次沒有獨立重跑。
