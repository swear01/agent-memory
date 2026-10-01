---
title: CVSD HW1 驗證、雙端繳交與 GitHub 同步
scope: projects/ntu-cvsd-adfp
status: submission unchanged; PR review findings accounted; grading and merge pending
updated: 2026-10-01
---

# CVSD HW1 完成紀錄

這份紀錄取代「HW1 尚未實作／formal 還在規劃」的舊狀態。繳交證據來自 2026-09-28，PR review 與來源不變的核對更新至 2026-10-01；後續查詢 PR、助教收件或成績時須重新查驗。

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
- HW1 原始碼／測試已推送 `add-hw1` 分支，原提交候選為 `983bfa20ad2e03d53beaafb24e48f91a9c4bccd5`；review 修正後 head `13020cda866de2d83d5a0049d650d27f367b4a40`；PR #1 尚未合併，不要把已 push 當成 main 已有 HW1。main 目前是種子 README。
- ADFP 啟動腳本在 `adfp-session` 分支，head `22c51bcbf2baac9aeadbcd9c9266b0e0b29f343e`；PR #2 尚未合併。保留兩個未合併工作樹：`<worktree-root>/CVSD2026/publish-hw1` 與 `<worktree-root>/CVSD2026/adfp-session`。
- 課程繳交與 GitHub PR 合併是不同狀態。下次先查既有繳交回執，不要因 PR 未合併或 OCR 把 1 看成 l 就重新提交、覆寫遠端包或修改已驗證 RTL。

## 2026-10-01 PR review 核查完成；無需重新繳交

- 已逐項回覆並 resolve 85 個討論串（HW1 49、ADFP 36）。resolve 數字代表核查處理，不等於 reviewer 原始 findings=0。PR #1/#2 保持 open，沒有合併。
- HW1 最新 `13020cd` 的 Swear **完整 PR review** 已完成；Gemini 同 head 的五項高優先級留言是等價符號延伸寫法。原生 VCS 比較 200,000 組（含每個 16-bit a 值、signed extrema、rounding 邊界）一致，未重現功能缺陷。其餘有效驗證問題已修正，規格外／時序誤報有下列證據。
- ADFP 最新 `22c51bc` 的 Gemini **完整 review** 於 2026-10-01 10:08:43 UTC 明確表示沒有需處理的留言，符合第一個 passing provider 的 OR policy。Swear 同 head 留言已逐項證實為既有 README 防護、原生誤報或環境假設建議。未把舊 head 的 pass 當成新版 pass。
- Swear `gate.mode=off` 的 SUCCESS 只證明執行完成；要看 findings。增量 0 findings 不能替代 full PR review。Codex 此輪明確回覆 review 額度耗盡；Cursor 未確認登入／repo 啟用，沒有把這些當 pass。
- **繳交用 `alu.v` 與 `rtl.f` 和 9/28 候選完全相同**，RTL SHA-256 仍為前述值。官方 testbench、patterns、lint.tcl 與 formal properties/run.tcl 也未改。修改的是不納入繳交包的 runner、oracle、測試、文件及 ADFP launcher，因此本輪不需要重打包或重繳 `cvsd_hw1_v1.tar.gz`，也未操作課程重新提交。

### 真問題修正與驗證補強

- Lint regex 原本可能把 10 fatals/errors 當成 0；改為只接受空白分隔的零計數。Injected 10 Fatals/10 Errors 都拒絕。
- 官方 mixed-atan2 verdict／reset 診斷存在漏判限制；runner 獨立拒絕 `[ERROR]`、`[FAIL]`、reset error／timeout，強制執行獨立 reset，並要求 atan2 MSE=0。官方 fixture 保留，不能把其真限制稱作誤報。
- VCS `$fatal` 可回傳 process exit 0。Runner 除 exit status 外也要求 PASS marker，缺少 marker 時失敗並印出 log；`tee` wrapper 用 pipefail 保留工具失敗。Wrapper 固定自身目錄，cleanup 不會刪到呼叫目錄。
- Oracle 的外部資料／常數審核由 assert 改為 ValueError，避免 `python -O` 跳過；stress 在 readmemh 前拒絕 N=0、超容量、unknown，零 Atan2 prefix 輸出 mse=not_applicable 而非除零。
- Reset bench 補上全部 reset registers；sixth 殘留的隔離 mutation 被拒絕。Formal XML mutation self-test 每次 flow 必跑，選擇確實有相關欄位與 implication 的 assertion，cover／assumption mutations 也須拒絕。
- ADFP 實測 inherited socket 有多個 listener PID，單次 multiline `ps -p` 會失敗；改為逐 PID 驗證。啟動固定已核對的 Chromium app 路徑，明確處理 FortiClient／Chromium launch 失敗。
- ADFP 補上 profile mode 0700 準備與失敗 gate、startup 提示與可定位的 UUID 診斷。`No service` 即使 exit 0 也拒絕，不開任何 app；此項是故障注入驗證的邊界防護，未宣稱本機 scutil 實際曾回傳這種組合。

### 已證偽的回報與證據範圍

- 規格保證 rotate b=0..16、Gaussian a∈[-1,1]；不得為 b=17..31 或超域 Gaussian 修改已驗證 RTL/oracle。實際 rotate datapath 窮舉 65,536×17=1,114,112 cases PASS。
- 規格第 9 點保證矩陣八個有效輸入完成後才發下一指令；中途 scalar 交錯不屬合法輸入。Back-to-back cover 實際可達：新 formal cover=covered；native busy=0、out_valid=1、accept=1、emitted=7，兩個矩陣共 16 outputs PASS。
- 官方 nonzero MSE 統計的 16-bit 差值可溢位，這是真限制。原生窮舉 65,536 個輸出確認 zero-MSE 仍只接受 exact golden；runner 要求 MSE=0，獨立 stress 逐筆 exact 比對，不會由此誤放行。
- macOS `install -d -m 700` 會把既有 0755 目錄收緊至 0700；native stale SingletonLock/SingletonSocket 可自動恢復且保留 profile marker，不需刪 profile。兩次真實 concurrent cold launch 都 exit 0，僅一個 Chromium owner；不加多餘 shell lock。
- README 已明示 loopback CDP 沒有認證、其他本機 processes/users 能控制 session，chmod 不等於 port 認證。僅在可信 Mac 使用，使用後關閉專用 browser；不要宣稱 0700 解決 CDP 認證。
- 最新完整回歸：I0–I10 PASS；兩輪 368,779 vectors，各 225,970 atan2 at quantized MSE=0；lint 0 fatals/0 errors/7 warnings；formal 19 proven、24 covered、17 non-vacuous、2 assumptions。Runner cases=11、wrapper cases=4、optimized-Python rejection cases=2、ADFP cases=14。
- ADFP native browser 檢查只 stub VPN=Connected，真實 VPN 為 Disconnected；未登入或重新提交 ADFP。沒有 GitHub EDA CI workflow，沒有新增 TA 收件或成績證據。
- Review 修正已 commit/push 並核對遠端 SHA；兩個原工作樹已 fast-forward 且乾淨。接手時讀 `<project-root>/outputs/CVSD2026-PR-review-audit.md` 與其 `verification/` 證據，再查最新 PR head，不要重啟已結束的 review 隊列。
