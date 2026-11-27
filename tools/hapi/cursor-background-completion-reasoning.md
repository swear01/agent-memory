---
title: HAPI shows Cursor background-completion reasoning in chat
scope: tools/hapi
status: verified
updated: 2026-11-28
tags: [hapi, cursor, acp, reasoning, notify_on_output]
source_refs:
  - https://github.com/tiann/hapi/issues/2267
  - https://github.com/tiann/hapi/pull/2268
  - https://github.com/tiann/hapi/pull/2354
  - hapi:cli/src/cursor/cursorAcpRemoteLauncher.ts
  - hapi:cli/src/cursor/postCloseHarnessSilence.ts
  - hapi:hub/src/config/sessionSummaryContract.ts
  - hapi:web/src/chat/normalizeAgent.ts
---

## 背景 shell 結束後，聊天多出「Briefly inform」推理

- 先讀原始 Hub row 的 session `metadata.flavor`。在 HAPI 0.35.0.1 的案例裡， Cursor ACP 存成 `content.type: codex`，這個 envelope 無法辨識 agent。全域搜尋可能找不到 reasoning，改搜可見的任務文字再讀前後訊息。Chat 的 **Collapse Reasoning** 只改顯示。
- HAPI #2268 要先辨識 `AGENT_NOTIFY_SUMMARY` 的 `status: done`，才抑制後續推理。#2354 讓 Cursor 在 **Emit status summaries** 開啟時收到 footer 規則。0.35.0.1 的預設為 off，已驗證案例也未開啟且上一輪無 footer，即使裝了修正也不會觸發這條路徑。
- Cursor ACP 的 completion action 可在原 `session/prompt` 回 `end_turn` 前發生。當時普通與背景的 text/thought 共用 presenter，沒有來源或 completion ID。用 prompt 狀態、shell 退出時點或文字相似度直接去重會吞正常回答，應依明確來源與穩定 terminal event ID 判斷。保留第一次成功／失敗、同一工作的新結果和 soft-steer 回覆。原案沒有 raw ACP stream，此結論來自本機 presenter 對照。

## 發送過、讀過、使用者看到，要各自驗證

- **前景已用 Read 看見 terminal 成功，仍收到回報。** 修法不能只改 Await。把前景 Await 或讀到受管理 terminal 的 confirmed final state 對應至同一 execution/event。任意檔案路徑或看似 terminal 的輸出不足以建立關聯。保留 pending，直到已寫入結構化 terminal 結果或使用者可見訊息已明確交代結果。
- **隊列 ack 不代表 Hub 落盤。** Shell progress 和 terminal completion 各有 wakeup ID；當時 Cursor 對 queued completion 做 batch drain，回合返回後才 ack，失敗可 requeue。合成測試可證明隊列行為，仍須讀原 Hub transcript 驗使用者可見結果。
- **只修一個入口。** Cursor 2026.11.22-e9042b2 的 interactive、print、ACP 共用 registry 實作但各自 drain。Interactive 有 Await suppression，print stream-json 發 `system/task_notification`，ACP 有獨立 presenter。Registry 實例與 conversation 範圍都要核對，同 shell ID 不能跨實例或對話去重。HAPI 的共用 AcpSdkBackend 也不表示所有 agent 都安裝 Cursor 的 post-done rule。

[完整原始紀錄](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/tools/hapi/cursor-background-completion-reasoning.md)
