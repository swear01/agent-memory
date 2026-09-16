---
title: "空字串與未設定的 shell 預設值語意不同"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: 7817c31f4c6bb0287b959299f86af73edff22ffcc023f2b56b2506be1eabab7c
---

# 空字串與未設定的 shell 預設值語意不同

助手發現 VGUIDE_SPEC 已設空字串，使用冒號減號預設值展開仍加回 default.spc，蓋過原本想使用 config 自帶 spec 的意圖；後續回報 argv 已不帶該選項。

依 wrapper 契約分清 unset 與明確空值，組 argv 後讀回真正參數。需要表示不傳選項時，不用會把空值再換成預設的展開方式。

來源只有 argv 修正的自述，沒有新實驗有效性結論。舊 worker 還在寫 log 的刪目錄競態是另一個生命週期問題。
