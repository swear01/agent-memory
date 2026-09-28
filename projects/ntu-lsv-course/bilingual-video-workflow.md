---
title: COOL course 65093 雙語影片教材與 Luna 翻譯流程
scope: projects/ntu-lsv-course
status: verified
updated: 2026-09-29
---

# 教材位置與閱讀

EEE5028 邏輯合成與驗證，第 1–2 週共五支錄影。教材放在 macOS Google Drive 的 `<drive-root>/document/學校講義/碩一/邏輯合成驗證`，週目錄為 `第01週_09-08`、`第02週_09-15`；單元資料夾按內容命名，不能只用錄影日期。根目錄保留首頁、總覽、兩週目錄與「資料說明」。本機預覽副本在 `<codex-documents>/2026-09-28/https-cool-ntu-edu-tw-courses/outputs/course`，其週目錄為 week01、week02。

# 可重複使用的流程

- 用 Codex 內建瀏覽器取得 COOL 播放器的 YouTube 網址；登入由使用者完成。不要控制或干擾個人瀏覽器。
- yt-dlp 下載原片與英文自動字幕。中文字幕回傳 429 不能證明沒有中文字幕；英文可用時先處理英文。
- 使用者指定 GPT‑6 Luna（gpt-6-luna）補上繁體中文對照及中文講義。每批最多 60 個 cue，用固定原始 ID 的 JSON map 回傳。相鄰字幕供理解上下文，但不可移動時間邊界或合併、遺漏段落。
- 檢查 ID 集合、非空中文、原英文與起迄時間完整保留；保留英文原檔。最初無索引的陣列曾出現段數及對齊錯誤，改用固定 ID map 才通過檢查。
- 抽查技術概念和畫面：固定變數順序下化簡有序 BDD 唯一；K-feasible cut 葉節點數最多 K；simulation 相同不能單獨證明等價；miter SAT 提供不等價反例，UNSAT 證明編碼範圍內等價。無法判定的原字幕標示「原字幕不清」。
- 中文優先閱讀，保留英文開關、雙語搜尋及影片時間跳轉。圖文逐字稿使用同一組主題截圖，完整字幕按時間配對；截圖不能代表全部板書變化。
- Agent 優先讀中文講義與 transcript.bilingual.json，再實際開相關截圖；需要精確原話、公式、程式碼時查英文、畫面及原片，回答附錄影與時間戳。
- Google Drive 更新文字與閱讀頁即可，不必重複複製大影片。檢查相對連結後將暫存移至垃圾桶；保留仍使用的預覽伺服器。

# 已驗證成果與限制

五支影片合計 3,201 段：882、904、1003、155、257；63 張主題截圖。新增 notes.zh-TW.md、transcript.bilingual.json/.txt、lecture.zh-TW.srt、context.bilingual.md，更新 reader.html/context.html 及中文總覽。

資料說明/check-bilingual.py 可重跑，核對英文、時間戳、中文字幕段數、講義圖片及本機連結。來源與 Google Drive 副本均通過。內建瀏覽器實測 260908-2 中文「可行」命中 3 段、英文 network 命中 12 段；英文開關可隱藏對照，時間按鈕跳至 3.52 秒，配圖页含 904 段與 14 圖。

譯文來自自動英文字幕，未重新做語音辨識，未人工逐字校正全片。使用現有 agent 額度，未另呼叫付費翻譯 API。Google Drive 本機副本已驗證，遠端雲端同步完成尚未獨立确认。不是任意課程一鍵自動下載的桌面應用程式。
