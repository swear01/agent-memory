---
title: CVSD HW2 雙處繳交與驗證收尾
scope: projects/ntu-cvsd-adfp
status: submitted-and-verified; grading pending
updated: 2026-10-13
---

# HW2 完成狀態

本紀錄取代 HW2 尚未實作、只有軟體排程估計或尚未繳交的舊狀態。
這是可公開的驗證摘要與操作經驗；完整作業、實際學號檔名、私人收據、原始
logs 與源碼留在私人課程 repository 的 `hw2/SUBMISSION.md`、`hw2/docs/rtl.md`
及 `hw2/verification/results/`，不因記憶同步而公開。

## 繳交與驗證證據

- 使用者明確授權後，2026-10-13 已完成 ADFP home 放置與 COOL v1 繳交。
  COOL 回執顯示 08:25（UTC+8）；下載已提交 tar 後與上傳檔逐 byte 相同，
  八個內容檔案亦逐檔吻合。ADFP 在 Linux 加入原始課程 SpyGlass report 後打包，
  server 解開最終 tar、要求九個檔案、比對整個 tree 並核對雜湊；完成 marker
  透過 Guacamole 與 in-memory OCR 讀回。
- 官方規則要求 Linux 打包 `studentid_hw2_vk.tar`、學號英文字母小寫，
  tar 放 ADFP home 並上傳 COOL。兩邊都要交；只有 COOL 可以省略 report。
  `01_rtl/` 必須含 file list、所有自寫 RTL 與 include header。助教提供的
  testbench／memory、DesignWare IP、材料、測資與 build outputs 不放入提交包。
- DVLab 與 ADFP 的公開四組測資皆通過完整 65,536-word Frame RAM 比對：
  p1 為 4,275 cycles，p2–p4 各 96,302 cycles。課程 VCS 2024.09-sp2、
  SpyGlass W-2024.09-SP2 為 0 fatal、0 errors、4 warnings、0 waivers。
  剩餘訊息已隔離至未讀 Frame read data 與原廠 DW／工具設定；不能宣稱 warning-free
  或所有 warnings 均已獲評分豁免。
- 目前 RTL 不使用 valid 前預讀。舊 4,005／30,765-cycle 結果不符合後續 Q&A，
  不能作為目前合規成績。軟體排程下界不等於實測 RTL、全體架構下界或滿分保證。
- 當前源碼與繳交紀錄都有 Codex 當前 head 的「沒有重大問題」結果，兩個 PR 已合併、
  私人課程 repo main 已與遠端同步，本輪 worktrees 已清理。
  Swear Review gate off 的 SUCCESS 只代表執行成功；0 findings 若同時註明
  `Review skipped: no items were selected` 也不能當成完整 review。此次文件 PR
  另取得 Codex 實際 review 才合併；額度耗盡或沒有回應不算 passing review。
- ADFP 放置與 COOL 回執不能證明助教已收檔、hidden tests 正確、lint 分數接受或
  最終成績。後續程式變更也不能推斷已重新繳交；先查私人紀錄與既有提交，
  不要未經新授權覆寫已驗證版本。

## 可重用操作經驗

- COOL 內建瀏覽器點 file input 沒開 chooser 時，已驗證可先
  `waitForEvent('filechooser')`，再對實際 file input 執行 `press('Enter')`，
  最後 `chooser.setFiles(...)`。已選檔案或「正在繳交」不是成功證據；讀回執並下載比對。
- ADFP 要同時驗證 native browser fullscreen 與 remote terminal／EDA fullscreen。
  遠端 shell 是 tcsh；Bash redirection 要用 `bash -c` 或上傳 Bash script。
  螢幕分析只用記憶體中的圖片，不把截圖或登入資料放進記憶 repository。
- 課程 `cb` 選到的 Python 曾因 `_Py_LegacyLocaleDetected` shared-library mismatch
  無法啟動。HW2 驗收腳本改用 Bash 解析 lint summaries，並以故障注入確認
  compile failure、缺少 PASS、缺少 summary 與多位數 errors 都會被拒絕，
  不修改共用課程環境。
- 官方補充 `hw2_models.tar` 是人類可讀的 vertex/index 文字資料。`teapot` 有
  696 頂點／880 三角形，對應 p1；`hutao` 有 25,484 頂點／32,110 三角形，
  對應 p2–p4。封存檔轉成 FP32／整數後，與公開 `.mem` 全部逐 word 相同。
  p2–p4 共用幾何與 projection，只改 view；補充資料可供理解與 reference，
  不需另讀進 RTL 或補交。這是封存官方檔案的比對，不是重新線上下載成功的聲稱。

## 記憶同步邊界

Codex 本機 ad-hoc 筆記及只索引該目錄的 collection 不是跨機器同步完成。
新增可公開的長期記憶後，整理進 canonical `agent-memory`，檢查公開可見性、
frontmatter、個資／秘密與完整 Git 歷史，commit／push 並核對遠端內容與 CI，
最後更新 QMD `memory` collection 並測試搜尋及讀回。QMD 的 SQLite、vectors、
models 與 cache 保持 host-local，不提交到 GitHub。私人來源繼續保留私人，
不要為補同步而把整個 Codex 記憶資料夾掛上公開 remote。
