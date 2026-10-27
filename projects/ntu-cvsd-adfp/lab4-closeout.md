---
title: CVSD Lab4 PrimeTime STA 與多 session 共用 ADFP 的教訓
scope: projects/ntu-cvsd-adfp
status: submitted-and-file-verified; grading unverified
updated: 2026-10-27
tags: [PrimeTime, STA, ADFP, Guacamole, COOL, Canvas, lock]
---

# Lab4 完成紀錄（2026-10-27）

- Lab4 是 PrimeTime 靜態時序分析：讀入課程提供的 `ALU_syn.v`、`ALU.spef`，以及 CBDK `typical/fast/slow.db`，並照原樣執行 `ALU_pt.tcl`，不修改。工具是 PrimeTime Y-2026.03，跑在 ADFP cvsd-vm-3。教材在 `Lab4.tar`；講義是 `115-1_Lab4.pdf`。只有被抽到的學生要交 PDF，截止為當天 20:10，不接受遲交。
- 13 題結果：
  - 14 references，總面積 1936.73。
  - 週期 10 ns、I/O delay 1.5 時，setup slack +6.92，critical path 是 `reg_B_reg[0]`→`alu_out_reg[7]`。
  - 週期 2 ns 為 −1.08 setup violation。最小週期 3.1 ns：另跑驗證，3.0 ns 為 −0.08，3.1 ns 為 +0.02。
  - output delay 8 時 slack +1.22，path 是 `alu_out_reg[0]`→`alu_out[0]`。
  - `report_constraint -all_violators` 有 8 個 hold violation，在 `alu_out_reg[0..7]/D`，最差 −0.24。`set_clock_uncertainty 0.0` 之後 212 checks 全部 MET。
- `report_timing` 預設只報 max path（setup），hold 違規要用 `report_constraint`／`report_analysis_coverage` 才看得到。uncertainty 歸零只是示範它會加在 hold 要求上，不是實際的 hold fix；實際修法是在短路徑插 delay buffer。
- 唯一的 Error 來自 script 第一行的 `remove_design -all`（尚未載入 design，SEL-005），不影響結果。
- 4 頁 PDF（答案表與 6 張終端截圖）在 17:20 Asia/Taipei 送到 COOL，attempt 1，`late=false`。回下載的檔案 SHA-256 為 `35042e141b411ed82745ccaa9fc18ff0c8f91f43609f3cb2d80b162f309d56ca`，與上傳檔相同。TA 驗收與成績未確認。

## COOL（Canvas）不經檔案選擇器的提交法

內建瀏覽器已登入 COOL，但讀不到本機檔案。改用 Canvas 的三段上傳：

1. 在 COOL 分頁 POST `/api/v1/courses/<c>/assignments/<a>/submissions/self/files`，body 帶 `{name,size,content_type}`。header `X-CSRF-Token` 取自 URL-decode 後的 `_csrf_token` cookie。回應會給 `/files_api` 的 upload_url 與已簽名的 `Policy`、`Signature`。
2. 本機 shell 用 `curl -F` 依序送出全部 upload_params，最後才送 `file=@...`，不需要 cookie。COOL 會回 302 到 `/api/v1/files/<id>/create_success?...&uuid=...`。
3. 在 COOL 分頁 GET 那個 create_success URL，再 POST `/submissions`，body 為 `{submission:{submission_type:'online_upload',file_ids:[id]}}`。

驗證：GET `submissions/self`，在分頁內 fetch attachment URL，用 `crypto.subtle.digest('SHA-256')` 計算後與本機 `shasum -a 256` 比對。

## 多個 session 共用同一個 Guacamole 桌面

- 所有 session 透過 CDP 9223 操作同一個 Guacamole 分頁，鍵盤只送到前景的 xfce 視窗。自己另開 cdp 腳本、不經 lock 就打字，會和 HW3 session 的 base64 heredoc 上傳交錯，造成指令亂碼與終端卡住。
- HW3 主 session 在其 scratchpad 的 `shared/` 提供 `adfp.sh`（`shot`/`run`/`put`/`wake`）。這個 lock 用 `mkdir lock.d` 加 `lock.d/pid`，只有 pid 已死才會被別人搶走。用 TaskStop 中止一個持有 lock 的指令，會留下沒有 pid 的 `lock.d`，所有等待者都會卡死。這次卡了約 9 分鐘，最後由持鎖方寫入一個存活背景迴圈的 pid 接管，才解開。
- 需要獨佔時（使用者裁定 Lab 優先於 HW3 sweep）：寫入長期存活的背景迴圈 pid 持鎖，先通知主 session，把對方的終端最小化（不要關），自己另開 Terminal Emulator 做事。結束時把對方視窗還原到 bash prompt，`rm -rf lock.d`，再通知對方。新開的 terminal 是 tcsh，要先 `cb` 再 `bash`。
- 終端看起來凍結時，滑鼠點 Applications 選單可以判斷遠端是否還活著；開一個新 terminal 比反覆送 Ctrl+C／Ctrl+Q 有效。
- 從 Guacamole 檔案瀏覽器用 click／dblclick 觸發下載，即使用 `Page.setDownloadBehavior`，檔案也沒有落地。讀 ADFP 輸出改用 `grep`／`sed -n` 分段顯示後截圖。
- headless Chromium `--print-to-pdf` 已寫出 PDF 但程序沒有結束。確認檔案存在後終止該程序即可。
