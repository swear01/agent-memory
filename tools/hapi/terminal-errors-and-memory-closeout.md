---
title: HAPI 終止錯誤與記憶收尾驗證
scope: tools/hapi
status: active
updated: 2026-09-06
---

# 在線狀態不能證明 turn 成功

HAPI inspect 的 active/thinking/lifecycle 與底層 Codex turn 結果是不同層。
已核對的案例中，peer 摘要没有顯示底層 rollout 的 event_msg/task_complete 錯誤；
session 仍在線，輪次卻已由 cyber_policy 中止。調查「停止工作」時應檢查
error.codex_error_info 和終止時間，再與實際程序、產物、job 結束狀態交叉核對。
一般 runner log 或只含 assistant 文字的摘要不足以排除 provider error。

cyber_policy 證明 provider 安全機制中止該輪，不證明是哪條命令觸發，也不證明
程式有惡意或修復成功。保留事件與既有產物，釐清合法工作範圍或走 provider
授權／誤判處理流程；不要自動重試或換 provider 繞過安全機制。

# 記憶收尾要分層驗證

Issue comment、canonical Markdown、Git commit/push、QMD 索引是不同交付。
每一層都需要自己的證據。看見記憶檔不代表 worker 更新過 QMD；qmd search
找得到內容也不代表 Git validation 通過或已同步 remote。

Fleet 共用記憶 checkout 時，由一個 coordinator 串行執行文件整併與同步；
workers 回報精簡、已驗證的經驗與交付證據，不同時 pull/push 或刷新同一索引。
QMD 設定與 cache 使用每台主機獨立路徑。歷史 worker 的 QMD receipt 若不存在，
應記為未確認，並對現在的 canonical 內容重新 update/search 驗證，不能補稱歷史成功。

關閉 session 前收集 final handoff、durable lesson 或無新增經驗的理由、memory
path 與驗證結果。若全庫 validation 被既有資料或 history 問題擋住，保存本次
文件並明列未 push；禁止把本地搜尋成功當成通過發布檢查。

使用者明確規定：每次 QMD 更新後都必須完成 canonical memory 的 Git 同步，
並讀回遠端 HEAD 確認；未同步只能回報待完成，不能宣稱收尾完成。
