---
title: TOEFL daily pages and precomputed teacher skill
scope: projects/toefl-practice
status: verified-with-limits
updated: 2026-10-15
---

# TOEFL 每日子頁與教師備課

## 現行需求與入口

- 本輪練習以寫作與口說為主；閱讀保留選用入口、不列缺交，聽力僅安排短複習。不自行加入完整四科模考或輕量版。
- 每日列表 → 獨立 Day 子頁 → 口說／寫作／選用閱讀。Day 1–14 口說維持完整 7 題複誦＋4 題訪談，暖身另列。
- 寫作作答頁沿用既有全科網頁的青綠頂欄、倒數、Back／Next、左右題目與作答區。Email 收件資訊與 Discussion 同學回覆在右側；只列當天課表安排的任務。
- 固定練習材料先備好，上課時不讓 AI 臨場另生整組。每天保留講評與本人訂正時間；通常口說一點、寫作一點，合計兩個優先問題。

## 檔案位置與執行

工作區是 `<personal-drive>/document/工作紀錄/碩一/交換申請/06_托福衝刺/`，以下路徑相對此根目錄。

- `00_使用導覽.md`、`02_目前程度與進度.md`、`03_練習紀錄.md` 為接續入口，再用 `01_托福衝刺時程.md` 與 `練習材料/說寫課表.json` 查實際課程。
- `練習材料/每日口說閱讀/開始每日練習.command` 啟動或沿用兩個 localhost 服務，首頁在 `127.0.0.1:8768/`，日次如 `/day/day02`；寫作路由 `/writing?day=day02&task=day2-discussion`。
- 口說／閱讀由 `127.0.0.1:8769/?day=day02&section=speaking` 或 `section=reading` 開啟；網址保存日次／科目，返回當天不能跳錯日。
- `作答與評估/寫作課程/` 與 `作答與評估/每日口說閱讀/` 是本人答案的本機位置。第一稿鎖定，訂正分開；每次口說重錄另存。不要把預備教材或範例當成已交作答。

## 教師 skill 與答案

- `教師備課/00_教師使用說明.md` 為可見索引；`.agents/skills/toefl-sprint-teacher/SKILL.md` 是 39 行入口，已接入工作區 AGENTS.md。
- 本機 Skillshare 原件為 `.skillshare/skills/toefl-sprint-teacher/`，專案 copy target 為 `.agents/skills/`。修改原件後執行 `skillshare sync -p`，不把此私人課程技能發佈成全域或遠端技能包。
- `references/manifest.json` 記來源 SHA-256、題庫版本與日次／寫作任務映射；只按當天讀 `dayNN.json` 及 `writing-<id>.json`，未選閱讀就略過 optionalReading。
- 備課涵蓋 15 組日次、101 題複誦原句、57 題訪談示範、12 篇寫作範例、7 組共 50 題組句與 255 個閱讀作答點。
- `references/rubrics.md` 依新版 ETS 說寫任務規準整理；`feedback.md` 提供短講評流程與虛構示例。老師資料不放作答網頁；兩個服務的相關路徑均驗證回 404。
- 訪談／作文示範不是唯一答案或已獲 ETS 評分的範例。組句合理替代語序須人工核對；補字替代詞須符合前綴、長度、語法與全文意思。
- 說寫任務可給 0–5 教學暫評，不能線性換成正式整科 1–6。缺件不是零分；ASR／逐字稿不能證明發音與韻律，未核聽時仍可提供內容與語言回饋。

## 驗證與邊界

- `python3 教師備課/檢查備課.py` 已通過：完整題目覆蓋、原題／標準鍵、組句詞塊、課表與網頁映射、來源版本及技能原件／copy 一致。
- skill frontmatter 驗證通過；安全稽核無 HIGH／CRITICAL，只有官方 ETS 來源連結的 LOW 提示，已核對為預期引用。
- `練習材料/寫作課程/day-pages-verification.json`、`練習材料/每日口說閱讀/verification.json` 及 `教師備課/驗證_20261015.json` 保留子頁、保存重載、錄音匯出、手機排版與教師隔離的本機證據。
- 原始匯入與既有本人檔案經雜湊核對保留。未做 GPT 6.1 評分一致性實驗、未新增可靠原音獨立核聽，也未驗證 Drive 遠端同步；測試音源不等於實體麥克風證據。
- 共享記憶只記教學／工具做法與相對位置；不提交原稿、私人得分、錄音、完整課程內容或個人帳號路徑。
