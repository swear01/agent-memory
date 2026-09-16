---
title: "既有 agent 程序不會自動取得新群組"
scope: tools/processes
status: active
updated: 2026-09-16
evidence_digest: 3c32a06114d0457a7f090f7125026bd98ef3ef427a8c6b4b146241133ce55008
---

# 既有 agent 程序不會自動取得新群組

歷史助手多次回報目前程序未含 Docker 存取所需群組，並遭 socket 拒絕；在另一個 shell 改群組不能證明這個 agent 已更新，後來的上傳亦非正式 manylinux 產線。

核對實際執行者及其父程序的 supplementary groups。需要刷新身分時使用已授權的登入或 runner 邊界，再在真正 worker 回讀群組與存取結果；只開 child session 不保證跳脫舊父程序的群組。

不保留私人帳號與群組值，也不要求為單次拒絕重啟無關服務。來源沒有正式重建完成的證據。
