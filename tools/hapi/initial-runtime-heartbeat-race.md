---
title: HAPI 初始 runtime 心跳與 atomic spawn 權限誤判
scope: tools/hapi
project: hapi
status: active
updated: 2026-10-18
---

# 初始 runtime 心跳與派工驗證

已核對上游 issue tiann/hapi#2053、上游 main `317e39a0bb9c768ce135f3fbe3157f9e3fefeebe`、待合併 PR #1771 的 `121d0c158f792fc1c8195ba977eb0034a2dcd22e`，以及維護版 v0.30.7.4 的 release manifest。

`AgentSessionBase` 建立後立即呼叫 `ApiSessionClient.keepAlive()`，但原實作使用 Socket.IO volatile emit；連線尚未完成時完整 runtime 可被丟棄。connect handler 的 regular `session-alive` 僅含 sid、time、thinking，Hub 因此可能先看到 active、稍後才收到 permissionMode。Socket.IO 官方說明 volatile 在底層 transport 未準備好時也可能丟棄，因此連線先完成的首次 runtime 也需要 regular transport。

Hub 契約中的 active 只是 transport liveness，不代表 runtime 或 agent readiness。修補前的 PR #1771 atomic spawn 在 `waitForSessionActive()` 後立即比對 selection，可能把尚未回報的 permissionMode 判成 `spawn_selection_mismatch` 並清理正確啟動的子程序。上游 main 沒有此 atomic API，不能宣稱 stock main 已重現相同 HTTP 502。

修補分兩層：共用 CLI 只保留最新 heartbeat snapshot，在初次連線及重連時 regular emit；若 connect 先完成，首次 runtime 也 regular emit，後續週期心跳維持 volatile。即使尚未 materialize，也應記住 snapshot，但不可因此連線或建立閒置 session。Hub atomic spawn 用既有總期限等待獨立回報的指定設定，再驗證與交付 remit；不可把 requested settings 寫入快取後當成已套用證據。

v0.30.7.4 已 carry #1771 的較早 head `5a0465f68c62bc88ee65bc1f03a0ff28d5fe1578`，不是等下一版才首次納入 #1771。後續 PR 修正不會自動出現在已發佈的 binary；下一次維護版需要重新審核並重建 carry。不要把 release manifest 中舊 head 的 clean verdict 當成新 head 已審查。

四個新增 CLI regressions 已在未修正上游程式全部失敗：runtime-before-connect、connect-before-runtime、斷線期間最新 snapshot、lazy materialization 前保留 runtime。此處紀錄的是 source/test 證據，不能當成已部署或 live-agent 派工驗證。


## #1771 的整合修補

2026-10-18，依操作者要求關閉獨立 PR #2054，將 CLI 修補與 Hub 等待防護一起納入既有 #1771。已正常推送候選 head `8d34df9eac82c789f80b92132ed9e7efe54043c1`；保留先前 head `121d0c158`，並合併 canonical upstream `317e39a0b`，沒有 force-push。本地 pre-push hook 要求最新 upstream ancestry，因此更新舊 feature PR 時要正常合併主線，不能繞過 hook 或以重建分支丟棄既有歷史。

單純等公開 Session 欄位等於 requested 值仍不夠：atomic reservation 已預填 model、effort、modelReasoningEffort，permission 也可能來自 persisted preference。`SessionCache.getReportedRuntime()` 因此只保留通過既有 stale-watermark 檢查的 CLI heartbeat 值；Hub 比對此獨立證據，再做原有完整 selection 驗證。分批 heartbeat 可累積缺少的欄位；`null` 是已回報值，`undefined` 才是未回報。結束、租約過期、新 activation 與刪除身份時清除證據；Hub restart 也必須等新 heartbeat，不能從 DB preferences 重建確認證據。

Runtime 等待與 transport/native readiness 共用原本 deadline。已回報不符立即 `spawn_selection_mismatch`；未回報到期是 `spawn_timeout`；等待期間結束是 `spawn_ended`。都走原本 compensating cleanup，未確認前不可交付 remit；不增加固定啟動睡眠、不延長期限，也不移除權限檢查。

四個 Hub integration regressions 在修補前 source 失敗：延遲完整回報、僅有預填 model/effort、等待中結束、以及 active wait 已用完同一個 deadline。另四例驗證真實 mismatch、明確 null、end/restart/expiry 清除與 stale heartbeat 不計證據。使用真實 in-memory Store/SyncEngine/SessionCache，僅 stub Runner 和 remit delivery 邊界。

候選 merged tree 的 root typecheck、CLI 3051、Hub 1469、Web 3349、shared 327、relay 118 測試通過；CLI 12 與 Hub 3 個既有 skip。Fixture 重新產生無漂移。這些是本地 source/test 證據；仍須確認新 head 的 CI 與 bot verdict，不能沿用舊 SHA 的綠燈，也不能稱為已發版、部署或實機派工驗證。
