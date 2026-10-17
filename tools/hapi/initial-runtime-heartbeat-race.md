---
title: HAPI 初始 runtime 心跳與 atomic spawn 權限誤判
scope: tools/hapi
project: hapi
status: active
updated: 2026-10-17
---

# 初始 runtime 心跳與派工驗證

已核對上游 issue tiann/hapi#2053、上游 main `317e39a0bb9c768ce135f3fbe3157f9e3fefeebe`、待合併 PR #1771 的 `121d0c158f792fc1c8195ba977eb0034a2dcd22e`，以及維護版 v0.30.7.4 的 release manifest。

`AgentSessionBase` 建立後立即呼叫 `ApiSessionClient.keepAlive()`，但原實作使用 Socket.IO volatile emit；連線尚未完成時完整 runtime 可被丟棄。connect handler 的 regular `session-alive` 僅含 sid、time、thinking，Hub 因此可能先看到 active、稍後才收到 permissionMode。Socket.IO 官方說明 volatile 在底層 transport 未準備好時也可能丟棄，因此連線先完成的首次 runtime 也需要 regular transport。

Hub 契約中的 active 只是 transport liveness，不代表 runtime 或 agent readiness。PR #1771 的 atomic spawn 在 `waitForSessionActive()` 後立即比對 selection，可能把尚未回報的 permissionMode 判成 `spawn_selection_mismatch` 並清理正確啟動的子程序。上游 main 沒有此 atomic API，不能宣稱 stock main 已重現相同 HTTP 502。

修補應分層：共用 CLI 只保留最新 heartbeat snapshot，在初次連線及重連時 regular emit；若 connect 先完成，首次 runtime 也 regular emit，後續週期心跳維持 volatile。即使尚未 materialize，也應記住 snapshot，但不可因此連線或建立閒置 session。Hub atomic spawn 則仍需用既有總期限等待獨立回報的指定設定，再驗證與交付 remit；不可把 requested settings 寫入快取後當成已套用證據。

v0.30.7.4 已 carry #1771 的較早 head `5a0465f68c62bc88ee65bc1f03a0ff28d5fe1578`，不是等下一版才首次納入 #1771。後續 PR 修正不會自動出現在已發佈的 binary；下一次維護版需要重新審核並重建 carry。不要把 release manifest 中舊 head 的 clean verdict 當成新 head 已審查。

四個新增 CLI regressions 已在未修正上游程式全部失敗：runtime-before-connect、connect-before-runtime、斷線期間最新 snapshot、lazy materialization 前保留 runtime。此處紀錄的是 source/test 證據，不能當成已部署或 live-agent 派工驗證。
