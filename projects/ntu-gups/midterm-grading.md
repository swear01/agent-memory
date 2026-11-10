---
title: GUPS3006 EDA 期中考批改：TA 給分標準、學號對應、COOL 發成績流程
scope: projects/ntu-gups
status: done
updated: 2026-11-10
tags:
  - ntu-gups
  - grading
  - ntu-cool
---

# GUPS3006 EDA 期中考批改（2026-11-10）

資料都在 Drive「GUPS助教」：`期中考_參考答案.md`（含配分與 Q4 範例圖）、`期中考_評分.md`、`期中考_成績.csv`、`期中考卷/`（掃描 PDF）。分數不放本庫。

## TA 給分標準（使用者實際調分後確認，可沿用到之後的考試）

- 核心答案對就接近滿分（「有寫出來就給滿」「答案有對先給分」）；答錯的部分照扣。
- **嘗試分只給方向對的作答**。概念錯、做錯題目（例如拿別的電路做）、照抄題目 → 0，不給「有寫就給 20%」。
- 方向對但錯在細節：照參考答案的扣分單位扣（例如一列錯扣 0.5），不要套用別題的「錯一個扣兩分」。
- 題目寫「explain and demonstrate」時，解釋本身有分，算在各方法的配分裡。
- 隱含但沒寫出來的可給部分分：例如 event-driven simulation 的 Scheduled events 全對，代表 L_A 其實算對了，空白的 L_A 仍給一點。
- 兩人分數不一致時，先看是哪一方違反標準；只是相對不公平就考慮調高另一方，不要一律往下修。
- 使用者喜歡逐格對照原卷（放大圖）再決定，結論要能指出卷面上哪一筆。

## 學號要對 COOL 名單

手寫學號 7 份錯 3 份（數字讀錯、師大學號尾碼 H 讀成 4）。發成績前一律用 COOL `/api/v1/courses/<c>/users?enrollment_type[]=student` 的 `sis_user_id`／`login_id` **依姓名**對應，再回頭修正 csv。師大學生的 login 是「學號數字加 h」接師大網域、`sis_user_id` 為 null。

## COOL 發成績（course `64904`）

- 作業「Midterm Score」assignment `429714`：`on_paper`、100 分、Assignments 群組，成績手動公布。每位學生一則英文留言（總分、Q1–Q16 各題分數、失分原因一句）＋掃描 PDF 附件。2026-11-10 寫入並讀回核對（分數、附件位元組數），`posted_at` 皆 null，等 TA 自己按 Post grades。
- 做法見 `domains/other/ntu-cool-api-access.md` 的「留言附件與未發布作業」。
- 留言內容與語言先問使用者（這次選：各題分數＋簡短原因、英文、先隱藏）；內部用的嚴格分、助教備註不可出現在留言。

## 批改工具的經驗

- 內建瀏覽器打不開 `file://`：用 `.claude/launch.json` 起 `python3 -m http.server` 開本機批改頁。
- 批改頁讓 TA 調分存在 localStorage，**不會自動進 csv**；要明講並在 TA 定案後寫回、讀回總分比對。調分前把頁面值備份，寫回後清掉會蓋過檔案的覆寫值。
- 「待檢查」清單（題、學號、TA 分、建議、原因、套用鈕）比在對話中逐條討論有效。

相關：[[hw2-abc-grading]]、[[ntu-cool-api-access]]
