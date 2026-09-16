---
title: "建立 session 的選項須有建立前可用的資源"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: bbc3a8f88e2e8d3292a6998816c8af846074ac4a9bfaa49979f67f7435a27320
---

# 建立 session 的選項須有建立前可用的資源

歷史調查指出 Pi model/effort descriptor 標成 session availability，create 頁尚無 session 可查，所以統一 permission UI 的 carry PR 並未解決建立時的模型選擇。

按畫面生命週期核對資料來源的 machine/session 層級及 launch payload。Descriptor 或部署中有該 PR，不代表每個頁面在需要時都取得到選項。

來源到設計與檢查 machine-level 資源為止，没有完成入口或 effort 傳遞；不保留當時 catalog 作目前能力清單。
