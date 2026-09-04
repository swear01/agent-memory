---
title: swop CLI PATH, authentication, QMD, and local DeepSeek routing
scope: machines/swop
machine: swop
status: active
confidence: high
created: 2026-09-04
updated: 2026-09-04
tags:
  - windows
  - path
  - github-cli
  - git-credential-manager
  - qmd
  - deepseek
  - gateway
  - pi
source_refs:
  - live:swop-2026-09-04
  - local:GitHub-CLI-2.100.0-help
  - local:QMD-2.8.3-status
generated_by: openai-codex/gpt-5.6-sol
---

# Long-running process PATH

`swop` 的 Node 24.19.0 是 WinGet portable install；Node、npm 與 npm-global commands（包含 QMD）位於 `<local-appdata>/Microsoft/WinGet/Packages/OpenJS.NodeJS.LTS_.../node-v24.19.0-win-x64`。GitHub CLI 位於 `<program-files>/GitHub CLI`。User `PATH` 已包含兩者，但既有 HAPI/Codex session 不會自動取得安裝後更新的 environment block。

因此 `Get-Command qmd` 或 `Get-Command gh` 在長時間存活的 session 內回報 missing，不等於工具未安裝。重裝前先檢查 User `PATH`、WinGet package ownership 與預期 executable 的絕對路徑；目前 session 可暫時 prepend 已驗證的安裝目錄，新 session 才會自然繼承持久設定。Node 升級後也要重查 versioned WinGet 路徑。

`agent-memory` 的 CI validator 會用 Python 讀取完整 Git history。Windows 預設 CP950 可能在內容檢查前先發生 `UnicodeDecodeError`；執行同一套 validator 時先設定 `PYTHONUTF8=1`。這是 verifier encoding 前置條件，不代表 Markdown 或 Git history 驗證失敗。

# GitHub authentication layers

Git Credential Manager 與 GitHub CLI 是兩個獨立的 credential consumer。GCM 已有 account、`git clone`／`git push` 成功，不代表 `gh auth status` 已登入。`gh auth login --with-token` 對 classic token 強制要求至少 `repo`、`read:org`、`gist`；若 GCM OAuth token 缺少 `read:org`，把它臨時設成 `GH_TOKEN` 可以完成有權限的 repo operation，但不會建立持久 `gh` login。

不要列印、手動複製或明文保存 GCM token。需要持久 `gh` 登入時，使用官方 device flow：`gh auth login --hostname github.com --git-protocol https --web --skip-ssh-key`，讓 GitHub CLI 把新 token 存入 Windows credential store；完成後以 `gh auth status`、`gh repo view` 與 Git credential helper config 分別驗證 CLI API 與 Git transport。

# QMD state and meaning

QMD 2.8.3 已安裝，npm registry 的 `latest` 也是 2.8.3。Machine-local `memory` collection 使用 `<temp>/qmd-memory-<host>/config/qmd` 與同根 cache；2026-09-04 的 `qmd update` 成功索引 94 份 Markdown，`qmd search model_catalog_json -c memory` 能找回 Codex allowlist 筆記。

QMD 不會自行「學習」聊天內容：canonical memory 是人工整理、驗證、commit 並同步的 Markdown，QMD 只負責 indexing／retrieval。此 Windows host 目前是 94 documents、0 vectors；lexical search 可用，但 vector／hybrid retrieval 尚不可宣稱健康。QMD 2.8.3 已是最新版，而 upstream Windows embed hang issue #679 仍 open；本機先前 CPU embed 長時間維持零 vectors，因此不要為了形式上的 closeout 再啟動相同長跑，直到 upstream 修正或有新的可驗證 workaround。

# Local DeepSeek gateway

`swop` 的 Pi `opencode-go` provider 指向同機 `127.0.0.1:35001/v1`，由 `DeepSeek Gateway (SWOP)` Scheduled Task 執行 Windows x64 gateway。Task 使用互動使用者 token、啟動時執行 `<home>/.local/bin/deepseek-gateway-safe.ps1`，並設有失敗重啟；從 OpenSSH session 註冊 task 時應以 `WindowsIdentity.GetCurrent().Name` 取得可映射的 local principal，不要使用可能只回傳 `WORKGROUP` 的 `USERDOMAIN`。

Gateway 設定與 route 分開存於 `<home>/.config/deepseek-gateway/config.yaml` 和 `routing.yaml`。五組 upstream credential 僅以 CurrentUser DPAPI blob 保存在 `<home>/.config/dvlab`，launcher 解密到自己的 Process environment 後啟動 gateway；User、Machine 與無關 Process environment 都維持 unset。Pi 的 `models.json` 只保存 `local-gateway` marker，不保存 upstream key。

2026-09-04 的 live verification 為 gateway `health=ok`、零 stderr、HAPI machine `pi-models` 正好八筆；Pi 經 strict runtime allowlist 呼叫 `opencode-go/deepseek-v4-flash` 與 `opencode-go/deepseek-v4-pro` 都回傳 `OK`。Flash 測試觸發 latch 向後切換到第三個 OpenCode endpoint，證明不是僅做 health check。HAPI Runner PID 與兩個既有 active Codex sessions 均保留，沒有重啟 Runner。
