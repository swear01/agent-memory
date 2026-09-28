---
title: CVSD HW1 驗證、雙端繳交與 GitHub 同步
scope: projects/ntu-cvsd-adfp
status: verified submission placement and COOL receipt; grading and PR merge pending
updated: 2026-09-28
---

# CVSD HW1 完成紀錄

這份紀錄取代「HW1 尚未實作／formal 還在規劃」的舊狀態。以下是 2026-09-28 已驗證結果；後續查詢 PR、助教收件或成績時須重新查驗。

## 繳交規則與證據

- 官方 `115-1_HW1.pdf` 第 5 頁指定 `cvsd_hw1_vk.tar.gz`，其中 `k=1,2,...`；第一版正確檔名是 **`cvsd_hw1_v1.tar.gz`**。截止時間為 2026-10-06 13:59:59 UTC+8。
- 包內只需 `<studentID-lowercase>_hw1/01_RTL/alu.v` 與 `rtl.f`，含必要資料夾。ADFP 與 COOL 都必須繳交：ADFP 在伺服器打包並放到 `<remote-home>/cvsd_hw1_v1.tar.gz`，COOL 上傳本機打包檔。
- ADFP 已完成放置；伺服器直接比較完整檔名、確認家目錄檔案位置、重新解出兩個檔案核對 SHA-256，並讀回 `RESULT FILENAME_AND_CONTENT_EXACT_PASS`。這是符合規定的放置證據，尚無助教自動收件回執或評分證據。
- COOL 以使用者指定的內建瀏覽器提交。回執顯示 2026-09-28 11:27 已繳交、檔名正確、大小 3.47 KB；從回執下載的實際檔案與本機候選包位元組相同，包內來源亦一致。教師尚未公佈成績。
- 兩端檔名、結構與包內內容相同；分別打包會有 archive metadata 差異，不要求兩端 tar.gz 整包雜湊相同。COOL 下載檔與本機候選檔的 SHA-256 是 `2701850630b6d502a0364915a05646affdbcd7cd9b12d5da5758bffcd891c315`。
- 已提交 RTL 的 SHA-256：`8a102776c16aa07baa2d247ad6f5d4799da9e45e0572064f7476f14be704ec2b`；filelist：`54e0ef2f0c52905184ee56011992514cc10705bb3bc6f37697b80255ef32f027`。官方 PDF 的 SHA-256：`618da6cbab7a02052c3015a91198ae432d4cf6ddf449df931fc45f41a0dda1e8`。

## 模擬、精度與 Formal 範圍

- 實驗室 VCS／SpyGlass X-2025.06、ADFP 2024.09-sp2：官方 I0–I10 全部 PASS，I10 量化 MSE=0；額外 368,779 筆，連續輸入與隨機間歇輸入均 0 mismatch，其中 225,970 筆 Atan2 的量化 MSE=0。這不是所有輸入正確性或隱藏測資排名證明。
- 最終 Atan2 採 64-bit XY/angle、60 個角度小數位與 56 次迭代；輸出 rounding 是 nearest、half tie 朝正無限大。精度比較需測四象限、軸線、最小負數與 half-LSB 分界，不能只看公開 40 筆或平均浮點誤差。
- SpyGlass：0 fatals、0 errors、7 warnings、2 infos；6 個 W415a 是靜態迴圈內多次賦值，另 1 個 CMD_overloadrule01 是工具規則設定。官方 testbench、patterns、rtl.f、lint.tcl 未修改，未新增 waiver。
- VC Formal Y-2026.03：19/19 assertions proven、24/24 covers reached、17/17 vacuity checks non-vacuous，2 個 assumptions（boot reset constant、legal opcode）。證明控制／狀態、reset、busy/input capture、MAC 保存、輸入輸出守恆、矩陣收集與輸出順序及完成時間；不是 Atan2／Gaussian 全輸入數值證明。
- VCS 中途非時脈邊緣 reset 測試：8 個非 idle 狀態、非零 MAC 清除、無殘留輸出與恢復均 PASS。隔離副本反轉矩陣欄位時，`p_matrix_order` 在 depth 10 找到反例；提交 RTL 未變。
- Formal 重跑用 `bash formal/run.sh`：容器執行 VC Formal，實驗室主機用 Python 檢查 XML；拒絕未證明、vacuous、timeout、停用與缺漏結果。完整本機證據位於 `<worktree-root>/cvsd-hw1/formal/formal/verified-results.txt`，原始報告在其 `logs/formal-verified/`。

## GitHub 與後續接手

- 私人 repo `swear01/CVSD2026` 已先完成 Gemini link；作業以最上層 `hw1/`、`hw2/` 等資料夾組織。不要把學生作業改成公開 repo。
- HW1 原始碼／測試已推送 `add-hw1` 分支，head `983bfa20ad2e03d53beaafb24e48f91a9c4bccd5`；PR #1 尚未合併，不要把已 push 當成 main 已有 HW1。main 目前是種子 README。
- ADFP 啟動腳本在 `adfp-session` 分支，PR #2 尚未合併。保留兩個未合併工作樹：`<worktree-root>/CVSD2026/publish-hw1` 與 `<worktree-root>/CVSD2026/adfp-session`。
- 課程繳交與 GitHub PR 合併是不同狀態。下次先查既有繳交回執，不要因 PR 未合併或 OCR 把 1 看成 l 就重新提交、覆寫遠端包或修改已驗證 RTL。
