---
title: "壓測殘留要先確認歸屬，不能把所有孤兒程序清掉"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 613304464877e88700b772b2b1acf6daf13bf35128d568cbfb92ba065a2e3a6a
---

# 壓測殘留要先確認歸屬，不能把所有孤兒程序清掉

歷史助手在壓測崩潰後回報高負載與大量孤兒，重跑時不同測試輪流逾時；進一步分類才分出本次暫存測試 worker 與仍被 fleet 追蹤的 session。

以程序身分、擁有者、啟動時間與工作狀態辨識本次可清理範圍，保留其他 session；先恢復可解釋的測試環境再重跑。孤兒、殭屍與忙碌程序並非同義，數量本身不證明 CPU 負載根因。來源沒有最終 gate 或清理驗證。
