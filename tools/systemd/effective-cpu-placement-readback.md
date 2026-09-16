---
title: "CPU 隔離以程序與 cgroup 有效值核對"
scope: tools/systemd
status: active
updated: 2026-09-16
evidence_digest: e93cbe72f5eb64d4c2caa1252e38efd90108c3618c1d33b3755c7e6340387365
---

# CPU 隔離以程序與 cgroup 有效值核對

歷史觀測顯示目標 user slice 的 EffectiveCPUs 仍涵蓋全部 CPU，助手準備等待管理員配置後重跑；相鄰 AppArmor 設定屬不同問題。

分開記錄設定意圖、manager 的有效 CPU 集合與實際 worker 放置。對實驗影響按具體觀測時間與既定資源契約判斷，不因檔案已寫或一次瞬時負載就宣稱隔離完成或全部結果受污染。

來源没有管理員修改後回讀，本次也未重設任何使用者的 CPU 或 AppArmor。已完成案例能否使用依可核對的受影響範圍決定，不一律丟弃。
