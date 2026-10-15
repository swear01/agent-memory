---
title: LSV 圖文影片教材、知識延伸與來源查核流程
scope: projects/ntu-lsv-course
status: verified
updated: 2026-10-15
---

# 教材位置與閱讀

目前主閱讀路線為第 1–6 週共 18 支影片、173 個圖文章節。教材位於 macOS Google Drive 的 `<drive-root>/document/學校講義/碩一/邏輯合成驗證`；根目錄只保留「原始講義」與「影片解析」，主入口為 `影片解析/index.html`。最新整理副本位於 `<codex-documents>/2026-10-15/users-swear-library-cloudstorage-googledrive-stanley/outputs/邏輯合成驗證`。下方 2026-09-29 的五片資料與處理數量為歷史紀錄，不能當成目前主路線。

# 可重複使用的流程

- 用 Codex 內建瀏覽器取得 COOL 播放器的 YouTube 網址；登入由使用者完成。不要控制或干擾個人瀏覽器。
- yt-dlp 下載原片與英文自動字幕。中文字幕回傳 429 不能證明沒有中文字幕；英文可用時先處理英文。
- 最初前兩週的逐 cue 翻譯曾由使用者指定 GPT‑6 Luna（gpt-6-luna）；後續新增影片採繁中章節解析，並非新增逐句中文譯稿。若處理原有逐句翻譯，每批最多 60 個 cue，用固定原始 ID 的 JSON map 回傳。相鄰字幕供理解上下文；此規則適用原字幕對照檔，不可移動時間邊界或合併、遺漏 cue。閱讀版另按下述方式合句。
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

PDF 文字擷取已證實會漏掉底線：PA1 第 2、4 頁實際命令是 `lsv_cut_tt <k>`、`lsv_cut_bddsize <k>`，不可依 pages.md 改成空格。命令／公式必須看原 PDF 圖像；三位 Luna 稽核建議須經來源複核後套用。

# 第二週原片優先（2026-09-29）

使用者指出教師出國的第二週不應採用錄影的錄影。內建瀏覽器核對 Week 1 announcement（COOL discussion_topics/524836）及兩個播放器；260915（G749faIQmWQ，10:19）的實際截圖顯示教室投影舊錄影，240910 ABC Tutorial（dBt7pnxLiXc，19:30）是 COOL 直接提供的舊原片。主要首頁／總覽改採後者；260915 設 reading_role=archived-rerecording，資料留存、不列主要路線。主閱讀四支／60章／385段，PA1要求另依2026 PDF；五支原字幕仍全量核查。來源與 Drive 本機副本檢查及內建首頁查核通過。

# 統一圖文章節版與延伸補充（2026-10-15）

- 使用者接受「概念問題 → 清晰來源圖 → 中文推導與例子 → 自測 → 原片時間／PDF 頁碼」，並要求舊片也統一版型。概念投影片優先用正式 PDF 配圖；板書與操作保留原片截圖，兩者標清來源。不得將 PDF 插圖說成錄影截圖，或把整理補充說成教師原話。
- 原有主路線四支、60 章、3,046 段英文 cues、385 段合句雙語逐字稿保留。第 3–6 週新增 14 支、113 章、12,362 段英文 cues，約 12.69 小時。全課目前 18 支／173 章／15,408 段原始英文；來源依既有兩份影片清單分開記錄。
- 原始講義索引已補齊 14 份 PDF、1 份 PPT 與專題題目文字檔；PDF 原件保持 byte-identical。講義抽字是搜尋用途，精確符號仍核對 PDF 圖像。保留 lecture02 舊版，主要閱讀採加上 QBF 的 lecture02_v1。
- 第 1–2 週保留 `transcript.bilingual.json`、`transcript.readable.json/.txt`、`lecture.en.srt`、`context.html`、`context.readable.md` 與全部截圖；這些來源在版型轉換前後逐檔 SHA-256 相同。其 `study-guide.json` 為章節 list；新增影片的檔案為含 `chapters` 的 object，讀取時先確認 schema。
- 新片完整英文自動字幕保存為 `captions.en.json3`，另有正規化的 `transcript.en.json/.txt`；原檔 hash、cue ID 與時間由 checker 驗證。原片媒體下載本次回傳 HTTP 403，不能宣稱已逐秒重看、取得新片截圖或復原所有板書；圖文閱讀與原片時間連結仍可使用。
- 新增 `延伸補充.html/.md` 與 `資料說明/延伸補充.json`。10 項主題涵蓋 BDD 跳層計數與 ordering、ZDD terminal、Tseitin output assertion／投影、incremental assumptions、QBF 量詞依賴、cofactor 真值表、care-set miter、prime／irredundant 非 minimum 的反例，以及 weighted column dominance。對應章節有可展開區，另有課程首頁入口。補充不另計為錄影章節。
- 數值例子以真值列舉、未帶 complemented edges 的普通 ROBDD／ZDD 建圖及精確 cover 枚舉核對；結果保存於 `資料說明/延伸補充驗證.json`。這不是執行外部 SAT solver 或效能 benchmark 的證據。
- 重跑 `python3 資料說明/check-bilingual.py`：核對舊雙語來源 hash、完整英文 cues／時間、章節覆蓋、配圖、講義原件、延伸補充映射與全部本機連結。內建瀏覽器實測搜尋、英文開關、自測、原片圖片放大、章節定位、雙語逐字稿入口及窄螢幕排版。原片的精確用語仍需按時間回看。
- 寫回 Drive 前在本機保留被修改檔案的備份，驗證來源沒有並行修改，再以內容 hash 確認寫入及 outputs 副本。Google Drive 本機內容已驗證，遠端雲端同步尚未獨立確認。
