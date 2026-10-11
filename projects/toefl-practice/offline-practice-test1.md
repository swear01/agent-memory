---
title: TOEFL 2026 Practice Test 1 全科離線作答練習完成紀錄
scope: projects/toefl-practice
status: verified-with-limits
updated: 2026-10-11
---

# TOEFL Practice Test 1 全科版

## 最終版本與延續入口

- 完成固定 Practice Test 1 的 97 個任務：Reading 40、Listening 34、Build a Sentence 10、寫作 2、Speaking 11。84 題可客觀計分；寫作與口說保留自行評估。
- 最終檔案位於 `<user-documents>/Codex/2026-10-10/new-chat/outputs/`：`TOEFL_2026_Practice_Test_1_全科作答練習.html`、`TOEFL_2026_Practice_Test_1_全科離線練習包.zip`、`全科版使用說明.txt`、`全科作答畫面預覽.png`、原始 `toefl-ibt-full-length-practice-test1.pdf`。
- HTML 自帶 43 個官方影音資源（34 個題目／提示檔、9 個說明檔），離線可用；ZIP 包含 HTML、說明與原始 PDF，已驗證解壓及內容位元組。
- 原始 `完整互動版`、`含官方影音` 與 `閱讀介面參考版` 均保留。後續以「全科作答練習」為準，勿把第一輪閱讀介面當作最終版。
- 原始碼基線在 `<task-root>/work/interface-source`；全科初版為 `task/official-interface` 的 `a74aac5`。2026-10-11 填字修正版在 `<worktree-root>/toefl-practice-<task-id>/reading-cloze-repair`，分支 `task/reading-cloze-repair`，commit `c3e0dd7`；最新麥克風修正版在同層 `microphone-session`，分支 `task/microphone-session`，commit `9585370`。核心檔案為 `practice.html`、`README.md`、`full-check.cjs`、`cloze-check.cjs`、`microphone-check.cjs`。此原始碼 repo 沒有 remote，只有本機提交，不能稱已推送或已有 PR。

## 官方參考與實作範圍

- 視覺與流程對照 ETS 的 `toefl-ibt-test-overview.pdf`（Reading 實體頁 5–6、Listening 8–11、Writing 13–16、Speaking 18、20）及官方 `student-practice-test-1-audio-files.zip`，題目與答案以原始 Practice Test 1 PDF 為準。
- 官方線上 sample 只觀察到歡迎與設備檢查畫面；沒有完成官方網站的麥克風檢查或整套線上作答，不能聲稱逐頁完全相同。
- Reading 支援同模組 Back／Next／Review，確認後鎖定模組。Listening 先播音訊再單題作答，不展示逐字稿、沒有 Back；長題組共用音訊。
- Writing 字庫可點擊、拖曳、交換與鍵盤操作；Email／Discussion 分欄編輯，含字數、剪貼與復原。剪貼簿權限不足時提示使用系統快捷鍵。
- Speaking 提示影音結束後直接錄音、時間到自動停止，IndexedDB 保存錄音；可逐段下載，並以 JSON／ZIP 備份及還原進度與錄音。
- 閱讀每模組 15 分鐘、聽力每題 30 秒、組句共 6 分鐘是練習設定。Email 7 分鐘、Discussion 10 分鐘、Interview 45 秒參考官方；Repeat 逐題 [8,8,10,10,10,12,12] 秒為練習配置，官方僅公開 8–12 秒範圍。
- 固定題本不具 ETS 適性題庫或校準後的 1–6 分數；不可把客觀題正確率當作正式 TOEFL 成績。

## 已驗證與未驗證

- `full-check.cjs` 通過 97 個任務、84 個客觀答案與錯答拒絕、23 次實際聽力播放、11 次實際 MediaRecorder 錄音；錄音輸入使用隔離的測試音源，沒有錄取使用者實體麥克風。
- Q1 8 秒與 Interview Q8 45 秒使用真實時間驗證停止；其餘錄音縮短計時驗證流程。11 段匯出錄音均經 ffmpeg 完整解碼並有非零測試音訊。
- 重載保留答案、作文、位置、暫停狀態與 11 段錄音；ZIP 匯出後在新瀏覽器 context 匯入、重載成功。舊 JSON、損壞音訊、無效 JSON、儲存空間不足、麥克風／自動播放拒絕、重新錄音拒絕保留原檔、逾時與延遲貼上均有測試。
- 桌面 1280×800、手機 390×844 無橫向溢出；深色系統下保持作答白底；全流程沒有外部 HTTP 請求或頁面 JS 錯誤。原始 43 影音、題目對應與 84 答案未改動。
- 證據保留於 `<task-root>/work/full-verification/verification.json`、`<task-root>/work/recording-verification/results.json`；測試備份及測試錄音不能冒充使用者答案或實體收音證據。
- 尚未測實體麥克風、Safari／Firefox；剪貼簿測試使用 stub，未碰使用者系統剪貼簿。最後已在原生 Brave 開啟並讀回最終頁面。

## 避免重犯的問題

- 原先 Reading Module 2 的藝術工作坊 email 缺正文，已依原始 PDF 實體頁 9 補回；後續比對固定題本時也要檢查段落正文，而非只計題數。
- 組句答案正規化移除標點後可能留下尾端空白，造成正確句子被判錯；`normSentence` 最後再 `.trim()`，以全 10 題答案驗證。
- CSS 的 `.btn { display:flex }` 曾蓋過原生 hidden 屬性，須保留 `[hidden]{display:none!important}`。逾時時要關閉尚未確認的對話框，避免確認後再跳一題。
- 匯入音訊須先完整驗證／解碼再變更狀態；IndexedDB 以 transaction 完成作為持久化成功證據；晚到的非同步貼上不得改動已逾時或切換的題目。


## 2026-10-11 填字題修復與答後估分要求

- 使用者要求作答留存，完成後由 AI 讀取完整備份，依 ETS 新版 rubric 分析作文、口說及客觀題型表現，提供 1–6 與 0–120 的診斷估分範圍。頁面沒有自動換分不等於拒絕 AI 評估；不可捏造這份固定題本的 ETS 原始分數等化表。
- Module 1 實際出現 Q1–10 全空白但已完成模組，起始畫面曾直接顯示 Q11。可重現原因是舊 `readingStep:1` 被當成新單元游標。尚未開始且填字全空白的閱讀模組，`normalizeState` 現在把位置回到 0；已開始或已完成的合法游標保留。
- 兩模組原先只有寬度不同的灰色輸入框，欠缺明確字母格。現在按缺字數繪製字母格／底線，提供相同 maxlength 與 aria-label。未填完按 Next 先警告；模組結束確認列出全部未作答題號。
- 原生 Brave 作答先暫停並匯出完整 ZIP，再解除受影響 Module 1 完成狀態並回到填字頁；既有答案與兩模組剩餘時間不變，維持暫停讓本人恢復。補作中斷紀錄留在 `<task-root>/outputs/TOEFL_填字修復作答紀錄_20261011.txt`，估分不能當作未中斷的標準施測。
- 原生頁面 `pagehide` 會 `save()`：直接改 localStorage 後 reload，可能被舊記憶體 state 覆寫。應重載新版後走既有 JSON 匯入流程，同步更新記憶體 state、儲存及畫面。
- `cloze-check.cjs` 通過兩模組 20 個字母格數、舊游標修復、跳題／模組漏答警告、原生打字與長度限制、重載、合法已開始游標保留、逾時關閉確認框、手機排版。另以實際備份在隔離瀏覽器驗證答案／計時保留；`full-check.cjs` 再次通過全 97 任務、84 客觀答案、23 播放與 11 錄音及完整備份流程。
- 同路徑 HTML 及完整離線 ZIP 已替換、ZIP 三檔內容逐位元讀回相符；題目、答案與全部內嵌媒體未改動。證據在 `<task-root>/work/cloze-verification/`、`<task-root>/work/cloze-full-verification/`。新增示意圖 `outputs/填字題修復畫面.png`。


## 2026-10-11 口說逐題重複要求麥克風權限修復

- 使用者回報每題都重問麥克風權限。可確認的程式原因是 `startSpeaking()` 每次呼叫 `getUserMedia()`，而 `MediaRecorder.onstop` 每次把所有 track `.stop()`；本機 HTML 不應依賴瀏覽器永久保留 file URL 權限來遮蓋這個流程問題。
- 同一口說階段沿用仍為 live 的 `speechStream`；提示播放與題間把 audio track `enabled=false` 靜音，真正錄音前再啟用。停止單題錄音只停止 MediaRecorder，不停止 stream。
- 離開口說、完成測驗、成功匯入、清除進度、pagehide，以及原有的初始化／裝置錯誤路徑會釋放連線。裝置已 ended 時重新申請；被拒絕仍可重試並保留原錄音。重新開頁或裝置中斷可能仍需允許一次，不應聲稱永久免權限或繞過瀏覽器設定。
- 新增 `microphone-check.cjs`：隔離 Brave 與測試音源，實際兩題 MediaRecorder 輸出只呼叫一次 getUserMedia；驗證提示／題間靜音、錄音期間啟用、舊檔位元組保留、裝置中斷重取、拒絕保留原檔，以及離開／完成／匯入／清除／pagehide 的 ended tracks。原本權限拒絕案例先模擬連線釋放再拒絕，維持原案例的語意。
- `full-check.cjs` 再次通過全 97 任務、84 客觀答案、23 次播放、11 段錄音及真實 8／45 秒截止、完整備份與拒絕處理；11 段新測試錄音皆完整解碼且非零音量。填字 regression 亦再次通過。題目／答案資料及全部內嵌媒體逐位元不變。
- 使用者原生 Brave 在目前錄音結束後才備份與更新同路徑 HTML，重載後讀回新版函式、答案／作文／位置／剩餘時間，以及既有錄音 SHA-256 一致。本機備份保留在 outputs，禁止把實際作答或聲音提交到 source／memory repo。normalizeState 會為舊時計欄位補 `paused:false`；驗證比較其等價語意，不把補預設欄位誤報成時間改變。
- 最終 HTML、使用說明與三檔離線 ZIP 已更新，解壓逐位元讀回相符。證據在 `<task-root>/work/microphone-verification/`、`microphone-full-verification/`、`microphone-cloze-verification/`。source 提交 `9585370` 工作樹乾淨，但仍無 remote；保留本機 worktree，沒有 PR／推送證據。
- 未代替使用者操作實體麥克風或權限對話框；自動化證據是 API 呼叫次數與假音源錄音，不是原生權限提示次數或實體裝置收音品質。實體使用需由本人接著作答觀察。


## 2026-10-11 答後診斷評分流程與音訊界線

- 使用者答完後，先從原生頁面匯出最新完整 ZIP，確認 finished／各 completed、客觀答案、兩篇作文及 11 段錄音齊全，再分析；不要求本人重新抄錄或重做。私人備份、報告、逐題明細及轉錄只留本機 outputs／work，禁止提交原始答案、分數或聲音到共享記憶／source repo。
- 客觀題要再次從原始 ETS PDF 的答案頁獨立核對；全題本 84 個鍵可用 pdftotext 與 DATA 逐項比對。組句只用字串相等可能誤拒合理語序，須區分「符合給定標準答案」與「文法可接受但官方接受狀態未知」，不要擅稱替代語序已獲 ETS 認可。
- 本輪文字介面的 audio helper 明確回覆 `audio content omitted because you do not support audio input`。不能宣稱實聽發音／語調；本機 ASR 與聲學時長／低音量間隔只支援轉錄內容診斷與完整性檢查，不能給需要原音的完整 rubric 題分。單一 ASR 會漏詞、改寫或同音誤辨；兩模型分歧要明示，不能直接當成發音錯誤扣分。
- 已用既有 whisper-cli、small-q5_1 與官方 whisper.cpp 模型來源的 large-v3-turbo-q5_0 作本機交叉辨識；先保留第一輪 txt／JSON，再跑第二輪，避免輸出覆寫失去比較證據。不向模型提供正確題句作引導，不把辨識字詞錯誤率當作 ETS 的評分公式。
- 每段原錄音轉為 16 kHz mono WAV，ffmpeg 解碼、非零音量、ffprobe 時長與兩版轉錄可作完整性證據；silencedetect 的低音量間隔不是已確認的猶豫，字數／短題提早停止也不應自動扣分。
- 依 ETS 現行 writing／speaking rubrics 作逐題診斷；完整口說題分需要原音評估。固定題本沒有已驗證的 raw-to-scale 表時，保留客觀正確率與可支持的寫作任務評分，不主觀指定四科分帶後套官方比較表。只有可靠的四科分數才可平均取最近 0.5 並解讀官方比較帶；沒有獨立預測誤差資料的範圍不能稱統計信賴區間。
- Email 的此份原始題本沒有明列至少 100 字，Discussion 有至少 100 字提示；評分須依個別任務原文，不能共用字數門檻。原本填字補作及麥克風中斷也須在診斷報告保留。
- 本機入口為 `outputs/TOEFL_Practice_Test_1_AI評分報告_20261011.txt`、`TOEFL_AI評分明細_20261011.json`、`TOEFL_口說辨識逐題對照_20261011.txt`；證據在 `work/scoring-20261011/`。記憶只記方法及位置，不複製本次私人得分與作答。


## 2026-10-11 官方規準對標與換分校正

- 使用者要求上網對準官方標準。已重新查閱 ETS 最新公開規準、教案、Test Overview 及 2026 尺度研究，並重新下載 Test Overview 與既有檔案 SHA-256 比對一致、查看實體評分頁。
- Test Overview 第 14–15 頁提供與 Practice Test 1 相同的詩刊投稿 Email 題及明確的 4 分示例；第 15–17 頁提供相同的強制志工 Discussion 題及明確的 5 分示例。這些是任務評分錨點，不能作原始分數換分表。官方高分示例仍有少量錯誤，須比較任務達成、展開及整體清楚程度，不能自訂逐字扣分。
- 2026 尺度研究第 12–13 頁說明寫作／口說先經標準設定，再以等百分位連結接到新版基準題本；閱讀／聽力使用 IRT 尺度連結。本次未找到固定 Practice Test 1 的可驗證原始換分表，不能借官方 0–120 比較帶替未校準的四科猜測背書。
- 已校正本機評分報告、JSON 明細及估分方法：撤回未校準的科分／總分及僅憑 ASR 推定的完整口說數字，保留有證據的客觀分數、寫作任務判斷、轉錄疑點與相鄰級別理由。原始完整 ZIP 雜湊不變；歷史 v1 留在 `<task-root>/work/scoring-20261011/superseded-v1/`，不要把它當最新結論。
- 證據補充在 `<task-root>/work/scoring-20261011/ets-overview-current.pdf`、`official-email-score4.png`、`official-discussion-score5.png`。本次方法校正不修改作答頁或原音，不保存私人評分於共享記憶。後續若有官方測評與練習配對資料，僅在驗證校準及誤差後給正式分數預測。

## 2026-10-11 明確要求六分制 AI 暫估與原音模型嘗試

- 使用者再次明確要求寫作、口說各一個新版六分制整科估分，並要求嘗試評原始語音。可以提供清楚標示為「主觀、未校準、可修訂」的診斷暫估；先前撤回的是未經驗證卻看似正式換算的分帶與總分，不能把校準限制誤解為一律拒絕程度判斷。
- 最新私人明細為 v3：`aiDiagnosticBands` 保存這類暫估及主觀範圍，`bands` 的官方等化結果仍為 null，口說完整任務題分／原始總和亦未恢復。範圍沒有統計涵蓋率，不能宣稱信賴區間、及格機率或必得分數；不由作文任務分按比例換六分，不搭配猜測的閱讀／聽力產生正式般的總分。
- 已在 `<task-root>/work/scoring-20261011/audio-evaluator-venv` 隔離安裝 mlx-vlm 0.7.6、MLX 0.32.3，從公開模型庫取得 `mlx-community/gemma-3n-E2B-it-4bit`，僅在 Mac 本機推論，沒有上傳錄音。模型的 feature extractor 預設截斷到 480000 個 16 kHz 樣本，即 30 秒；四段 45 秒 Interview 需切成 25 秒加餘段，合計七個完整 Repeat 與八個 Interview 片段。
- 首次 Metal GPU timeout，改 `prefill_step_size=32` 後完成全部音訊輸入與靜音控制。但第一輪評語指令多只得到轉錄，並出現空輸出、反覆生成及指令污染。另改成音訊在前、只求語音評語的四個 probe，靜音時捏造反覆字元，且把語音說成亂碼文字，未通過可靠度檢查。不能把「模型接收過原音」當成可靠的發音／語調／理解費力程度評分；本輪未採用其評語。證據保留於 `<task-root>/work/scoring-20261011/audio-evaluation/`。
- 原始完整備份雜湊不變，11 段解壓 WebM 與原 ZIP 逐位元相同。修訂前現存報告／JSON／方法保留在 `superseded-v2`；文字報告當時仍含舊版數字，因此以最新 v3 為準，不因歷史目錄名稱推定內容已校正。實際得分、文字作答、聲音及模型逐段輸出僅留私人本機檔案，不提交共享記憶。
