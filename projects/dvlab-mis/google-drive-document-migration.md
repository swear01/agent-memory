---
title: DVLab MIS HackMD to Google Docs migration
scope: projects/dvlab-mis
project: dvlab-mis
tags: [hackmd, google-drive, google-docs, documentation]
status: active
created: 2026-08-25
updated: 2026-09-16
---

# DVLab MIS HackMD to Google Docs migration

## 正式架構

- 私有網管文件的正式入口是共用網管帳號所擁有、維持 `Restricted` 的 `DVLab-MIS` Google Drive 資料夾。
- 2026-08 遷移當時是 19 份獨立正式文件加 1 份 `00 - DVLab MIS 文件索引`，全部為 Google 原生文件；索引取代 HackMD tags。這是歷史數量，後續新增服務文件應查即時清單。
- 2026-08-25 追加納入 `DVLab 本地 LLM：Valkyrie NInfer + Zeus Bifrost` 與 `DVLab 伺服器無主帳號清理公告`。
- 2026-09-11 起：canonical 已在 Google Drive。使用者確認後，HackMD 只保留給實驗室同學看的公開文件；其餘網管／私有 note 已用 CLI `notes delete`／`team-notes delete` 丟進垃圾桶（可還原，不是清空垃圾桶）。
- 仍留在 HackMD 的公開／成員文件：`Notes from DVLab MIS Team`、`DVLab Server Usage Policy`、`DVLab VPN 安裝指南`、`DVLab personal profile instruction`、`DVLab 伺服器無主帳號清理公告`、`GPU and CUDA`。
- `GPU and CUDA` 在內部目錄標成給成員看，但未登入訪客抓取為 forbidden；其餘五份訪客可讀。
- `DVLab MIS` team workspace 在清理後已沒有 note。私有網管文件以 Google Drive 為準，不要再把 HackMD 當回滾來源。
- 遷移當下的 6 個 Google 試轉產物先前已移至 Drive 垃圾桶，與這次 HackMD 清理無關。

## 已驗證的轉換流程

- Markdown 經 Pandoc 轉為 rich HTML，再貼入 Google 文件；貼入前移除 HTML `<title>`，並把文件基準樣式設為「一般文字」。
- 相對 `.md` 連結依固定 Google file ID 對照表改成直接文件連結。完成後索引有 19 個 Google 文件連結，未解析 `.md` 連結為 0。
- Drive 最終清單為 20/20 Google 原生文件，兩份追加文件各只有一份，沒有 Word 或 Markdown 副本。
- Google Docs 匯出 DOCX 回讀保留中文、標題、表格與等寬程式碼；表格分隔線的純破折號正規化不算正文遺失。
- 原始 18 份文件有 `HackMD 遷移基準 2026-08（驗證完成）`；兩份追加文件與更新後索引有 `HackMD 遷移擴充 2026-08-25（驗證完成）`。
- 批次匯入時暫時啟用的「將上傳的檔案轉換成 Google 文件編輯器格式」已恢復為關閉。

## 日常寫回（2026-09-11 已驗證）

- 本機預設 `gws` 登入的是個人 Google 帳號，**不是**網管共用帳號。寫 `DVLab-MIS` 前先用 rclone remote `dvlab-drive` 打 Drive `about.user`，確認顯示名稱是共用網管帳號後，才把該 remote 的 access token 拿去打 Docs API。
- 正常修改用 Docs API 增量更新（`replaceAllText` / 在既有清單項換行處 `insertText`），加 `writeControl.requiredRevisionId`；寫完立刻 export `text/plain` 讀回。不要用 Markdown 全文覆蓋，不要建同名副本。
- 2026-09-11 SSID 切換已寫回 `353 network setup manual(private)` 與 `Notes from DVLab MIS Team`：實驗室 Wi-Fi 只剩 ASUS 的 `DVLab353`，R9000 無線關閉，舊名 `DVLab353-2` 停用。

## 操作邊界

- 每次操作必須以 Google 畫面顯示的帳號身分確認共用網管帳號，不可把 URL 的 `u/N` 當成固定身分。
- 更新既有文件時依 file ID 寫回並讀回驗證，不建立同名副本。
- 記憶不得保存密碼、API key、文件正文、帳號資料、HackMD note ID、Google file ID 或受限文件 URL；精確對照以受控遷移包的 mapping 檔為準。

## 文件歸位與格式（2026-09-16 使用者糾正）

- 「文件要整理，不要全部放同一份」不是為每台主機另建一本混合筆記。先讀 `DVLab MIS Notes(private)` 的用途導覽及各正式文件現有章節，再按原架構更新。
- `DVLab Overview` → 設備、硬體規格、本機救援管理帳號及受限帳密紀錄；實體位置未確認時不得猜測設備區。
- `353 network setup manual(private)` → `device MAC and IP bindings` 內的主機紀錄，以及 `Firewall configuration` 內的 NAT／port forwarding。不要在文件頂端堆一段涵蓋所有主題的更新紀錄。
- `New Server Setup(private)` → 主機加入 NIS／NFS 的流程、實作結果、救援路徑與尚未完成的切換。新主機案例放在現有適用章節。
- Minecraft 這類獨立服務可有專用 runbook，集中版本、遊戲設定、白名單、服務操作、備份與還原；網路與主機資料連回原文件。
- 索引沿用「原生項目符號＋文件名稱本身帶 hyperlink」。不要寫成「名稱：完整 URL」，不要把裸網址或 Markdown 星號塞進原生 bullet。文字連結、段落樣式和項目符號必須一致；只有真實連結使用藍色底線，一般說明與標題不應繼承連結外觀，並清除空白 bullet。
- Docs `insertText` 會繼承插入點的 paragraph/list/text style；只比對 plain-text export 不足以驗收。回讀並檢查 `paragraphStyle`、`bullet`、`textStyle.link`，確認標題層級及內容位置；必要時匯出 PDF 做視覺驗證。
- 調整分類前保留受限快照；先寫入正確目的地並核對內容、權限，再移除錯位重複段落。舊文件連結可保留精簡導引，不能留下第二份帳密或設定來源。
- 使用者要求更新記憶與 QMD 時，必須真正完成 Markdown、同步、索引、embedding 及搜尋驗證，不能只更新 Google 文件便宣稱已記住。
