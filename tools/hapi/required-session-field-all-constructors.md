---
title: "Session 新 required 欄位要追到 production 建構處"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: a27b9dc3a168165a32c4b1f3d29ed36ac8ad2bff6b36f51054cbcb465a9c5ec8
---

# Session 新 required 欄位要追到 production 建構處

歷史 protocol resolution 修正後，typecheck 仍缺 serviceTier；助手先怀疑同名型別是第二份套件，檢查後改判 required 欄位漏加，且不只 fixture，production API 也直接建立 Session。

列舉所有 literal、schema 解析與 consumer，依完整契約同步欄位，不把錯誤文字當 duplicate-package 根因，也不只在 test 補值。來源到定位 production 與 fixtures 為止，沒有最終全套通過；先前 symlink 問題與後續 constructor 缺欄位應分開記。
