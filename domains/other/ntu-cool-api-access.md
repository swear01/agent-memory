---
title: "NTU COOL（Canvas）API 存取與隱藏權杖 UI"
scope: "domains/other"
status: active
confidence: high
updated: 2026-11-13
tags:
  - ntu-cool
  - canvas
  - api
  - access-token
  - file-upload
  - built-in-browser
---

# NTU COOL API 和檔案操作陷阱

- 研究所與大學部帳號、課程和 token 狀態獨立，數位系統設計用大學部帳號。先用現有授權的 `https://cool.ntu.edu.tw/api/v1` 核對 `users/self/profile`，再選課程／作業。不要假定任何主機已存 token；舊部署筆記不能當 live 憑證來源。API 不可用再走 Codex 內建瀏覽器；Brave 等外部瀏覽器只在使用者指定或授權時使用。COOL API 不靠 VPN；校內 ADFS 等入口依當時連線設定。
- 以授權介面申請或撤銷 token，不存正文、log 或截圖。頁面同源的 API request 可使用現有登入，Canvas 的 JSON 有時以 `while(1);` 前綴。HttpOnly cookie 無法從 JS 讀取，也不應移到另一台。頁面 API helper 以 `startsWith('/api')` 判斷完整 API path，避免把課程相對 `/assignments` 打到站根路徑。SSO 的真實欄位是頂層頁面，公告 iframe 和過期 snapshot ref 勿混用。
- 課程 Files 為空也可能有教材。查 `/courses/<id>/modules?include[]=items` 的 ExternalUrl 和 `external_url`，一般 `url`／`html_url` 可能只是 Canvas redirect。授權下載用登入分頁或 API；若以 API 為該檔取得帶 verifier 的臨時下載連結，也當成私有資料，不留 log。PDF 章節頁序應由渲染或原文確認。

2026 年數位系統設計的 course id 是 `45284`，大學部帳號。已下載教材存於 `gdrive:document/學校講義/大二/數位系統設計`。API 404 的舊檔曾標記 hidden_for_user，未下載不能說全量；Box 傳檔失聯時曾改核准 Drive 中繼再 rclone 拉取。[2026-09-29 個別教材、下載排除及傳檔收據](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/domains/other/ntu-cool-dsd-45284-materials.md)

## 上傳和繳交

1. 驗課程、作業、檔案與既有回執。Browser 缺 file chooser 時用 Canvas 官方三步檔案上傳：先 POST 至作業 `submissions/self/files` 取得 `upload_url`、`upload_params`，再用核准本機上傳介面送 multipart。Upload fields 放在 file 前，實際欄位名依回傳確認；`on_duplicate: rename` 可能在 display name 加 `-1`，不等同檔案內容改變。
2. 若返回 302，GET `create_success` Location 才完成，拿到 file ID、大小與 `upload_status: success`。漏 GET 時，file ID 可一直 404；舊簽名 policy 重 POST 可 400，從第一步重建。在獲准登入的路徑完整下載對本機 SHA-256。簽名參數和重導向 URL 不落共享記憶。
3. 上傳和繳交是不同操作。已明確要求繳交該次作業時，以 `POST /courses/<c>/assignments/<a>/submissions` 提交 `online_upload` 和 file_ids，驗 201、submitted、late 欄及回讀原附件。不要以按鈕變化或已存在 file ID 當作繳交證據。發文、寄信、發布作業或改設定依各寫入授權。

## 助教評分及留言

- 對學生用該作業 `submissions/<user_id>/comments/files` 上傳留言附件，再 PUT submission comment 的 `file_ids`。GET 帶 `include[]=submission_comments` 驗附件。輸入 `posted_grade` 後讀回 score，`post_manually` 下 `posted_at` 仍可為 null，學生還看不到。未發布作業曾拒絕打分 403；正式發布會讓學生看到、可能通知，不能為測試寫入私自打開。區分已儲存分數、已發布分數、留言附件和學期正式成績。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/domains/other/ntu-cool-api-access.md)
