---
title: LSV 講義與影片解析：來源、逐句對齊及行內補充
scope: projects/ntu-lsv-course
status: verified
updated: 2026-10-30
---

- **影片解析引用了錄影的錄影，或把 PDF 內容當教師逐字原話。** 正式入口為課程 Drive 的 `影片解析/index.html` 與 `原始講義`，依 COOL 模組對來源。第二週優先教師直接提供的舊原片，教室重錄標 `reading_role=archived-rerecording`；PA1 要求依 2026 PDF。以內建瀏覽器讀來源，正式術語查 PDF 圖面、原英文 cues 與相符 timecode。抽字曾遺失底線，實際命令為 `lsv_cut_tt <k>`／`lsv_cut_bddsize <k>`。
- **逐句翻譯遺漏／錯置 cue，或閱讀版被硬切句。** 上游英文原檔、ID、起迄保留。逐 cue 檔按最多 60 個原 ID 的 JSON map 回譯，驗完整 ID／非空中文／時間。閱讀版才按完整語意合句，`source_cue_ids` 完整回指 `transcript.bilingual.json`；原字幕不清明標。新增影片走中文章節＋完整英文 cues，不把英文自動字幕或編輯稿標成人工逐字校正。傳給整理者一併附影片、原 cue、PDF 頁與畫面。
- **章節補充只有摘要，或內容移到另開的頁面。** 用「概念問題 → 來源圖 → 中文推導與例子 → 自測 → 原片／PDF 位置」。投影片概念優先官方 PDF，板書／操作則用已核對的原片截圖。補充在對應章解說欄直接顯示，先「講義依據」附檔名頁碼，再「延伸推導與算例」。自製內容另標，獨立頁僅索引。讀 `study-guide.json` 確認舊 list／新 `chapters` object；更新同步調整 `資料說明/延伸補充.json` 的 anchors。
- **字幕或教材可讀就當全部媒體已抓到。** 429／403 只證當次抓取失敗。以 `build-reader.py`／`build-english-reading.py` 建頁，再用 `check-bilingual.py`、`test-reading.py`、`check-reading-pages.py` 查 source hashes、cue 覆蓋、相對連結、圖片、補充 anchors。另實測搜尋、原片定位、雙語和窄螢幕。Drive 寫入前備份、核對並行修改，完成以 hash 驗改動；本機 Drive 複本確認與遠端同步分開。可保存的原片按本機 `Movies/LSV/` 分流，不重複拷到 Drive。

[完整原文與實測紀錄](https://github.com/swear01/agent-memory/blob/a90d5791102e82884f021bff1b4feb7d74c35247/projects/ntu-lsv-course/bilingual-video-workflow.md)
