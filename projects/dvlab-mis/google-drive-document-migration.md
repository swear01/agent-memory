---
title: DVLab MIS HackMD to Google Docs migration
scope: projects/dvlab-mis
project: dvlab-mis
tags: [hackmd, google-drive, google-docs, documentation]
status: active
created: 2026-08-25
updated: 2026-08-25
---

# DVLab MIS HackMD to Google Docs migration

## 正式架構

- 私有網管文件的正式入口是共用網管帳號所擁有、維持 `Restricted` 的 `DVLab-MIS` Google Drive 資料夾。
- 最終集合是 19 份獨立正式文件加 1 份 `00 - DVLab MIS 文件索引`，全部為 Google 原生文件；索引取代 HackMD tags。
- 2026-08-25 追加納入 `DVLab 本地 LLM：Valkyrie NInfer + Zeus Bifrost` 與 `DVLab 伺服器無主帳號清理公告`。
- HackMD 原件沒有刪除或改寫，暫作回滾來源；公開文件仍可留在 HackMD。6 個試轉產物已在使用者明確確認後移至垃圾桶。

## 已驗證的轉換流程

- Markdown 經 Pandoc 轉為 rich HTML，再貼入 Google 文件；貼入前移除 HTML `<title>`，並把文件基準樣式設為「一般文字」。
- 相對 `.md` 連結依固定 Google file ID 對照表改成直接文件連結。完成後索引有 19 個 Google 文件連結，未解析 `.md` 連結為 0。
- Drive 最終清單為 20/20 Google 原生文件，兩份追加文件各只有一份，沒有 Word 或 Markdown 副本。
- Google Docs 匯出 DOCX 回讀保留中文、標題、表格與等寬程式碼；表格分隔線的純破折號正規化不算正文遺失。
- 原始 18 份文件有 `HackMD 遷移基準 2026-08（驗證完成）`；兩份追加文件與更新後索引有 `HackMD 遷移擴充 2026-08-25（驗證完成）`。
- 批次匯入時暫時啟用的「將上傳的檔案轉換成 Google 文件編輯器格式」已恢復為關閉。

## 操作邊界

- 每次操作必須以 Google 畫面顯示的帳號身分確認共用網管帳號，不可把 URL 的 `u/N` 當成固定身分。
- 更新既有文件時依 file ID 寫回並讀回驗證，不建立同名副本。
- 記憶不得保存密碼、API key、文件正文、帳號資料、HackMD note ID、Google file ID 或受限文件 URL；精確對照以受控遷移包的 mapping 檔為準。
