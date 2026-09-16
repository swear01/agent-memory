---
title: "Pattern kill 可能先終止清理 shell 自己"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: bf2fdd62ab74a1fb7a2a32d3bce461e01347f96a9f62b955150cfbe81f7a9ec9
---

# Pattern kill 可能先終止清理 shell 自己

歷史助手回報 pkill 使執行清理的 shell 退出，後續仍有程序存活，於是改用明確 PID；原文將問題定位為 pattern 自我匹配。

不要把整段清理命令可能包含的字串直接當成廣泛終止條件。先辨識本次擁有的程序及穩定身分，逐項處置並讀回剩餘狀態；命令中途退出不能算清理完成。來源沒有最終清理驗證，也不授權終止其他 session 的 child。
