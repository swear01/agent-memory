---
title: DVLab MIS Google Docs ownership and write-back
scope: projects/dvlab-mis
project: dvlab-mis
tags: [hackmd, google-drive, google-docs, documentation]
status: active
created: 2026-08-25
updated: 2026-09-28
---

# DVLab MIS 文件歸位與寫回

- 私有網管資料以受限 `DVLab-MIS` Drive 資料夾和原生 Google Docs 索引為準。HackMD 保留面向成員／公開的文件。讀取前以實際權限核對。
- 本機預設 `gws` 是個人帳號，先用受控 rclone remote `dvlab-drive` 查 Drive `about.user`，再核對目標資料夾和文件的擁有者。顯示名稱或 URL `u/N` 不能證明身分。file ID 對照見受控遷移包，憑證、受限文件 URL 和正文不放記憶。
- 依既有 file ID 用 Docs API 增量修改，帶 `writeControl.requiredRevisionId`。`replaceAllText` 做精準置換，新增索引項時在既有清單項換行處 `insertText`。避免全文覆蓋或同名副本。
- 寫後匯出 `text/plain`，正規化 CRLF 再比對。新增文字會繼承樣式，另讀 `paragraphStyle`、`bullet`、`textStyle.link`。索引使用原生 bullet 和帶連結的文件名，移除空 bullet 及標題／說明誤承繼的連結。必要時預覽 PDF。
- 歸位先讀 `DVLab MIS Notes(private)`。設備規格及受限救援帳密放 `DVLab Overview`；MAC/IP binding、NAT port forwarding 各放 `353 network setup manual(private)` 專節；NIS/NFS、救援及未完成切換放 `New Server Setup(private)`。獨立服務可用專用 runbook 連回主機／網路文件。先留受限快照並核對目的地內容與權限，再移除來源的重複段落。
- 還需轉舊 Markdown 時，Pandoc rich HTML 去掉 `<title>`、以一般文字樣式貼入 Google Docs，按受控 file ID mapping 替換相對連結，匯出 DOCX 檢查標題、表格、等寬程式碼。
