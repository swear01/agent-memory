---
title: "加入長按操作時保留原生 click 契約"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 593a020532a0c1d6e621efb6eb8730ecb535e0e75d6aa415cea2240c636901de
---

# 加入長按操作時保留原生 click 契約

原始測試輸出顯示 project row 在 fireEvent.click 後仍保持展開；歷史助手指出自己把 toggle 改接到長按 hook 的 mouseup 路徑，並提出保留原生 onClick、長按只開選單。

分別驗證一般 click、鍵盤啟動與長按是否只觸發預期動作；不要只為適應新的事件路徑而放寬舊行為測試。menuOpen guard 是當次建議，仍需核對 state 更新時序與雙重觸發。來源没有修後通過證據。
