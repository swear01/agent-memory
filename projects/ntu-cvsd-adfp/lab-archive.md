---
title: CVSD Lab1 Lab2 source and submission archive
scope: projects/ntu-cvsd-adfp
status: verified archive; existing submissions unchanged
updated: 2026-10-06
---

# CVSD Lab1 / Lab2 archive

## 保存規則

- 使用者要求 CVSD 作業歸檔的子資料夾使用英文，排除大型且非必要的檔案。完成成果不能只留在臨時 Codex 工作區。
- Git 保存最上層 `lab1/`、`lab2/`：RTL、testbench、file list、必要測試資料、驗證腳本與簡短 README。
- 作業區位於 `<GoogleDrive-root>/document/學習紀錄/碩一/電腦輔助積體電路設計/`。各 Lab 使用 `lab1/` 或 `lab2/`，底下分成 `materials/`、`submission/`、`evidence/`。
- `materials/` 放老師原始講義及小型 `Lab1.tar`、`Lab2.tar`；`submission/` 保留原封不動的提交 PDF；`evidence/` 放繳交收據與必要模擬、波形截圖。各 Lab 有 `SHA256SUMS.txt`，作業區根目錄有來源版本索引。
- Lab2 的 `Lab2_alu_s.v`、`Lab2_alu_s.sdf` 是老師提供、重跑實驗必需的輸入，合計不到 80 KB，應進 Git；不要用「生成檔」規則一併排除。製程 cell library 只記環境需求與既有引用，不複製進 repo。
- 不額外收錄大型 FSDB/VCD、simulator binary、cache、PDF 預覽或臨時傳輸包。Git 忽略這些執行輸出，但保留 Lab1 的 `program_out.txt` 標準答案。這次沒有刪除原始檔或遠端成果。

## 2026-10-06 已驗證結果

- 私人 repo `swear01/CVSD2026` 的 PR #3 已合併至 `main`。來源 commit 為 `5093bd73f16fa1d9d545037e61fbb5b26e379142`，merge commit 為 `5342b068d54f7c5b55a41661466a967588d9e158`。本機 main 與遠端一致且乾淨，本次工作樹與分支已清理。
- 17 個匯入的程式／驗證檔約 107 KB，逐檔與原有來源完全相同；教材輸入也核對原始 tar 的包內內容。GitHub main 上 21 個新增或更新的檔案均以 Git blob 身分核對。
- 作業區 14 個檔案共 7,469,206 bytes，已透過指定帳號的 Drive API 核對雲端名稱、大小、MD5 與 SHA-256，皆與本機副本一致；不僅是 File Provider 路徑出現檔案。
- Lab1 的 Git RTL 已補 `X` 宣告並修正減法；另外保存 `Lab1_alu.original.v`。Lab2 保存六個原始教材檔及 RTL/gate 兩份修改後 testbench，修改僅為 waveform dump / SDF 設定。
- 保留的 `verify_adfp.sh` 是原本當次使用的批次腳本，需要 ADFP home 中的指定輸入，並建立固定日期目錄；README 已交代這些前提，不把它當成可無條件重複執行的通用 runner。
- 本次只做歸檔與內容、shell syntax、README link、ignore-rule 檢查，沒有改寫 RTL、重跑 EDA 或重新提交作業。

## 提交版本與證據邊界

- `<studentID>_lab1.pdf` SHA-256：`483650001307ca0d4e8fde42c732a37b2c92dcbc1c6a8ba6323c5357889b16c6`。
- `<studentID>_lab2.pdf` SHA-256：`3ff55854c575c3c06f8cfba121820681f5dbb1e90e666e4b9cdb0d051f4572e9`。
- 兩份提交 PDF 都與既有下載副本一致。保存的 Lab1 收據顯示 2026-09-26 15:34 Asia/Taipei，標記逾期；Lab2 收據顯示 2026-10-06 19:16 已繳交。
- Git/Drive 歸檔完成不是 TA 現場驗收或成績證據。不要因為歸檔、PR 狀態或檔案整理而重新繳交。這次未從 ADFP 取回 raw waveforms 或 simulation logs。

後續接手先查最新 repo / Drive 狀態，以上 SHA 與數量是本次 verified snapshot，不代表未來版本。
