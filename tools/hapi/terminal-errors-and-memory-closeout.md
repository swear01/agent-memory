---
title: HAPI 終止錯誤與記憶收尾驗證
scope: tools/hapi
status: active
updated: 2026-09-07
---

# 在線狀態不能證明 turn 成功

HAPI inspect 的 active/thinking/lifecycle 與底層 Codex turn 結果是不同層。
模型也要用實際 turn_context 核對：已驗證案例中，worker 自述 Sol/high，
但兩筆 turn_context 均為 gpt-5.6-luna/medium，與 Hub 設定一致。全域
config.toml 的預設值和 agent 自我描述不能取代該輪 model/effort 證據。
已核對的案例中，peer 摘要没有顯示底層 rollout 的 event_msg/task_complete 錯誤；
session 仍在線，輪次卻已由 cyber_policy 中止。調查「停止工作」時應檢查
error.codex_error_info 和終止時間，再與實際程序、產物、job 結束狀態交叉核對。
一般 runner log 或只含 assistant 文字的摘要不足以排除 provider error。

cyber_policy 證明 provider 安全機制中止該輪，不證明是哪條命令觸發，也不證明
程式有惡意或修復成功。保留事件與既有產物，釐清合法工作範圍或走 provider
授權／誤判處理流程；不要自動重試或換 provider 繞過安全機制。

# 記憶收尾要分層驗證

2026-09-07 使用者明確指定 HAPI Codex 派工以 YOLO 啟動，避免一般 shell／
檔案操作反覆要求人工核對。新 session 明確傳 `--permission-mode yolo`；
已在執行的 Codex session 可透過 HAPI `/permissions yolo` 切換，讀回
`preferredPermissionMode` 與命令處理回覆確認，不只看送出成功。保留原模型、
medium/high 成本偏好與 Fast off；不把 YOLO 解讀為重試 provider 安全 halt 的授權。

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

使用者另明確要求：每次關閉 session 前，都檢查是否有應保存的新知，以及是否
真的已寫入。由 coordinator 核對實際 Markdown、validation、commit、QMD
update/search 與遠端 HEAD；worker 的「稍後更新」或單獨的口頭完成聲明不足。
沒有新知時記錄理由並引用既有已驗證內容，不為了收尾製造重複記憶。

已交接且没有可立即執行任務的 worker，不因 issue／PR 尚未結案或「可能還會
用到」而無限保留在線。把未完成項目、產物與恢復條件交接到 issue，再 archive
並 inspect 確認；關閉 session 不等於 issue 完成，也不刪除未合併 worktree。
受工具批准或外部依賴阻塞的 session 要明列 blocker、負責 issue 與恢復條件，
不能只依 thinking=true 算作有效進度，也不能無限重試來維持在線。

派工文字要明確指出「你已是 worker，直接執行，不再建立 session」。曾有 worker
把接手 remit 誤當再派工要求；發現後先停掉多餘協調輪次，確認唯一產物作者，
由 root 接管實際 worker 並收尾多餘 session，避免重複寫檔與無限等待。

# Jobs 能力與版本查核

2026-09-06 的 HAPI 0.29.0.6 fleet notes 明確記錄：operator 暫時排除
session-attached Jobs，CLI/API/meters/pinning 不提供，legacy schema/rows 保留。
因此舊 executable 有 job 子命令、DB 有舊紀錄，都不能證明目前 Hub 支援 Jobs。
先查目前 help 與部署註記，再判斷 HTTP404；不可僅憑404宣稱是升級故障。
本次只以 help/parser 查驗，未執行背景工作；若操作規範仍強制 session job，
其長工作要等支援契約恢復，不能用假 heartbeat 或旁路 launcher 冒充。

## Coordinator 持續管理（使用者明示，2026-09-07）

派出 sessions 不是協調工作的結束。核心 coordinator 要持續巡檢目前派出的
workers、驗收實際 artifacts、把不成立的 PASS 退回同一任務修正，並接續已
授權且條件具體的工作。完成的 worker 在 memory/QMD/Git 驗收後立即封存，
不能只列一張正在跑的表就留下全部 idle。新的獨立工作用 fresh session／
issue；原任務驗收修正可回到該 worker。Open session、完成 handoff、通過
驗收、PR merged、issue closed 都是不同狀態，必須逐一判定。
