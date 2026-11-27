---
title: HAPI 初始 runtime 心跳與 atomic spawn 權限誤判
scope: tools/hapi
project: hapi
status: active
updated: 2026-10-18
---

## Active 比實際 runtime 設定早到

在 PR #1771 的 0.30.x atomic spawn，`keepAlive()` 的初始 runtime 用 Socket.IO volatile emit，transport 尚不可用時可丟失；connect 後的 regular heartbeat 只有 sid/time/thinking。Hub 因此可能先看到 active，尚不知道實際 permissionMode，誤判 `spawn_selection_mismatch`。該路徑當時不在 stock upstream main；0.35 的介面見 [peer 指令對照](peer-cli-0-35-command-changes.md)。

- CLI 保留最新 runtime snapshot，首次 runtime、初次連線與重連用 regular emit，後續週期維持 volatile。Lazy session 在 materialize 之前也要記住 snapshot，但不能因此連線或建閒置 session。
- Hub 預填 model/effort 或從 DB 讀 permission 不等於 runtime 已套用。用 `SessionCache.getReportedRuntime()` 讀過 stale-watermark 檢查的 CLI 心跳，累積分批報到。明確 `null` 是已回報，`undefined` 是未回報；end、lease expiry、activation、delete 與 Hub restart 後不能沿用舊證據。
- 等待與 transport/native ready 共用同一 deadline。已回報但不合是 `spawn_selection_mismatch`，未報到期是 `spawn_timeout`，中途結束是 `spawn_ended`。這些都走原 compensating cleanup，未確認前不交付 remit。不要以固定 sleep 拉長窗口。
- 回歸要覆蓋 runtime/connect 先後兩種順序、重連、lazy state、只預填設定、真 mismatch/null、過期或重啟和已耗盡的 deadline。當時 candidate CI 和整合測試有驗證；0.30.7.4 已 carry #1771 更早 head，不能以 release version 推定含這組後續修正。

候選版本與完整驗證範圍見 `git show 2505b5e:tools/hapi/initial-runtime-heartbeat-race.md`。
