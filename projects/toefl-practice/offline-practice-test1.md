---
title: TOEFL 2026 Practice Test 1 全科離線作答練習完成紀錄
scope: projects/toefl-practice
status: verified-with-limits
updated: 2026-10-10
---

# TOEFL Practice Test 1 全科版

## 最終版本與延續入口

- 完成固定 Practice Test 1 的 97 個任務：Reading 40、Listening 34、Build a Sentence 10、寫作 2、Speaking 11。84 題可客觀計分；寫作與口說保留自行評估。
- 最終檔案位於 `<user-documents>/Codex/2026-10-10/new-chat/outputs/`：`TOEFL_2026_Practice_Test_1_全科作答練習.html`、`TOEFL_2026_Practice_Test_1_全科離線練習包.zip`、`全科版使用說明.txt`、`全科作答畫面預覽.png`、原始 `toefl-ibt-full-length-practice-test1.pdf`。
- HTML 自帶 43 個官方影音資源（34 個題目／提示檔、9 個說明檔），離線可用；ZIP 包含 HTML、說明與原始 PDF，已驗證解壓及內容位元組。
- 原始 `完整互動版`、`含官方影音` 與 `閱讀介面參考版` 均保留。後續以「全科作答練習」為準，勿把第一輪閱讀介面當作最終版。
- 原始碼基線在 `<task-root>/work/interface-source`；保留的 task worktree 在 `<worktree-root>/toefl-practice-<task-id>/interface-reference`，分支 `task/official-interface`，最終 commit `a74aac5`。核心檔案為 `practice.html`、`README.md`、`full-check.cjs`。此原始碼 repo 沒有 remote，只有本機提交，不能稱已推送或已有 PR。

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
