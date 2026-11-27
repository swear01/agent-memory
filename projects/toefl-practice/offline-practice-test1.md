---
title: TOEFL Practice Test 1 離線練習的跳題、麥克風與診斷評分陷阱
scope: projects/toefl-practice
status: verified-with-limits
updated: 2026-10-11
---

- **繼續原離線版時選到早期閱讀稿。** 正式入口是 `<user-documents>/Codex/2026-10-10/new-chat/outputs/` 的「全科作答練習」HTML／ZIP；原始碼只在本機。試題、可客觀計分的 84 個答案和 43 份媒體對原 ETS PDF／原音逐項核對，含閱讀正文。練習計時和 ETS 公開規定分開。組句正規化移除標點後再 trim；可接受的替代句法另外標示，不自行視為官方答案。
- **一開題本就跳過填字。** 舊 `readingStep:1` 可能被誤解成新單元游標。只在未開始且填字全空白時重設到第 0 步，保留合法進度與計時。先暫停匯出完整 ZIP，再重載新版用既有匯入更新 JS state、IndexedDB 和畫面；直接改 localStorage 會被舊 `pagehide` 尚在記憶體的版本覆蓋。未答先提醒，逾時須關確認框；過期的剪貼簿回呼不可再修改答案。
- **口說每題都重新詢問權限。** 同一口說區段共用仍為 live 的 `speechStream`，提示和題間靜音，錄音再啟用；單題停止只停 MediaRecorder。離開／完成／匯入成功／清除／pagehide 釋放，裝置 ended 再請求。拒權限要保留原錄音；音訊完整驗證後才匯入，以 IndexedDB transaction 完成為成功。跑 `cloze-check.cjs`、`microphone-check.cjs`、`full-check.cjs`；實測是隔離 Brave、測試音源與真 8／45 秒截止，未證實體麥克風、權限提示次數或 Safari／Firefox。CSS 的 `.btn` 不可覆蓋 `[hidden]`。
- **固定練習卷的正確率被轉成正式分數。** 匯出完整 ZIP 後按原題及相符 rubric 診斷。Test Overview 同題 Email 4 分及 Discussion 5 分示例是 task anchors，非原始分數等化表；字數要求依原題不同。可依使用者要求給寫作／口說各一個明標主觀、未校準的診斷暫估，勿把未校準四科猜測加成正式總分或信賴區間。
- **ASR、靜音檢查或原音模型接收成功被當發音評分。** 先驗音檔可解碼、時長、非空，再看兩份不受標準答案提示的轉錄；模型分歧與低音量只能標疑點。Gemma 音訊 extractor 預設截 30 秒，45 秒題需保全後段；即使分段完整，本輪模型在靜音控制有幻覺，評語未採用。用最新私人 v3 評分明細的 `aiDiagnosticBands`，官方等化 `bands` 仍為 null。完整口說任務分需要可靠原音；作答、分數與錄音留在私人本機。

[完整原文與實測紀錄](https://github.com/swear01/agent-memory/blob/93786b66aa982bda81d4c1ebe6cfceaa832b5d32/projects/toefl-practice/offline-practice-test1.md)
