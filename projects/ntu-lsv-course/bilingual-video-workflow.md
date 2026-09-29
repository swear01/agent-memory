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
- 使用者指定 GPT‑6 Luna（gpt-6-luna）補上繁體中文對照及中文講義。每批最多 60 個 cue，用固定原始 ID 的 JSON map 回傳。相鄰字幕供理解上下文；此規則適用原字幕對照檔，不可移動時間邊界或合併、遺漏 cue。閱讀版另按下述方式合句。
- 檢查 ID 集合、非空中文、原英文與起迄時間完整保留；保留英文原檔。最初無索引的陣列曾出現段數及對齊錯誤，改用固定 ID map 才通過檢查。
- 抽查技術概念和畫面：固定變數順序下化簡有序 BDD 唯一；K-feasible cut 葉節點數最多 K；simulation 相同不能單獨證明等價；miter SAT 提供不等價反例，UNSAT 證明編碼範圍內等價。無法判定的原字幕標示「原字幕不清」。
- 中文優先閱讀，保留英文開關、雙語搜尋及影片時間跳轉。圖文逐字稿使用同一組主題截圖，閱讀段落按時間配對；截圖不能代表全部板書變化。
- Agent 優先讀擴充中文講義與 transcript.readable.json，再依 source_cue_ids 回查 transcript.bilingual.json，再實際開相關截圖；需要精確原話、公式、程式碼時查英文、畫面及原片，回答附錄影與時間戳。
- Google Drive 更新文字與閱讀頁即可，不必重複複製大影片。檢查相對連結後將暫存移至垃圾桶；保留仍使用的預覽伺服器。

# 已驗證成果與限制

五支影片合計 3,201 段：882、904、1003、155、257；66 張閱讀版主題截圖。新增 notes.zh-TW.md、transcript.bilingual.json/.txt、lecture.zh-TW.srt、context.bilingual.md，更新 reader.html/context.html 及中文總覽。

資料說明/check-bilingual.py 可重跑，核對英文、時間戳、中文字幕段數、講義圖片及本機連結。來源與 Google Drive 副本均通過。早期逐 cue 頁面曾實測 260908-2 中文「可行」命中 3 段、英文 network 命中 12 段；英文開關可隱藏對照，時間按鈕跳至 3.52 秒，配圖頁含 904 段與 14 圖。

譯文來自自動英文字幕，未重新做語音辨識，未人工逐字校正全片。使用現有 agent 額度，未另呼叫付費翻譯 API。Google Drive 本機副本已驗證，遠端雲端同步完成尚未獨立確認。不是任意課程一鍵自動下載的桌面應用程式。

# 圖文閱讀版修正（2026-09-29）

- 使用者要求首頁成為可獨立閱讀的完整圖文講義，不能只留短摘要。現有五單元共 66 章，補足概念、推導、例子及截圖解說；包含原先漏選的電晶體、版圖、ABC BLIF/network 畫面。
- 圖片與解說各占半欄，圖片在左、文字在右；內建瀏覽器面板寬 582 與桌面 1400 均已實測等寬，390 寬改成上下排列。點圖放大，Escape 關閉。
- 原 YouTube cue 是顯示碎片。Luna 整理語意完整的中英句子／短段落，Codex 核對術語、投影片與斷句；共 401 段，source_cue_ids 連續涵蓋全部 3,201 個原 ID。不得只按 cue 數、固定長度或圖片切換硬切句子。英文閱讀版是編輯稿，精確原話回查原字幕。
- study-guide.json、transcript.readable.json 為可保存的整理資料。資料說明/build-reader.py 使用現有 Pandoc 與原生 CSS/dialog 渲染；不自行翻譯新影片，不新增服務或付費 API。原 JSON、SRT、影片不覆寫。
- SAT off-set 圈 bc¬d 必須整體取反，得到 ¬b∨¬c∨d；不要直接把 cube literals 改成 OR。來源 CNF 圖可核對 0110 與 1110。cut hexadecimal 顯示由 MSB 到 LSB 對應全 1 到全 0，LSB 對應全 0；不可與輸入枚舉順序混淆。
- 來源及 Drive 本機副本通過 check-bilingual.py（原英文/時間保留、閱讀段落覆蓋、圖片和連結）。內建瀏覽器 260908-2 的 BDD 搜尋命中 11/102 段、可行 1/102；英文開關、原字幕展開、時間按鈕與 URL?t 跳轉已驗證。遠端 Drive 同步完成仍未獨立驗證。
- 整理方法參考 lecture-to-notes、course2md、video-note-maker。course2md 的 pause/length 合併不能保證完整句意，須額外做語意重組與技術校對。原字幕不清之處保留限制，未重新做 ASR。

# 原始 COOL 講義作為 subagent 輸入（2026-09-29）

製作逐字稿與課綱前，先用 Codex 內建瀏覽器下載同課程／單元的教師原始 PDF（不搬 cookie，不接管個人瀏覽器）。已取得 lecture01-intro.pdf（26頁）、LSV26 syllabus.pdf（2頁）、ls-handout.pdf（112頁，2010背景教材）、lecture01-abc.pdf（46頁）、ABC_tutorial.pdf（39頁）及 LSV_2026_pa1.pdf（5頁）。合計六份／230頁；按既有週次與單元放入「原始講義」，共用講義放週目錄。資料說明/原始講義索引.json/.md 保存來源、SHA-256、頁數與影片對照；*.pages.md 供搜尋，圖／公式看原 PDF。

給 subagent 的輸入須包括影片、穩定 cue ID 原字幕、對應 PDF／頁碼及畫面，先核對術語與章節覆蓋，再整理完整句子與圖文講義；修正回報須附 PDF頁碼與錄影時間。PDF證明書面內容，不代表每個字都曾講出；未口述內容標「原始講義補充」。三個 Luna subagent 已做重點抽查，並修正 homework 遲交規則為每天扣20%、補 Mask Level、限定 RTL while 可合成性與 AIG/FRAIG 表述。原始字幕不覆寫。來源與 Drive 本機副本的PDF magic/hash、頁碼抽字、來源連結及原cue檢查通過；遠端 Drive同步未獨立確認。
