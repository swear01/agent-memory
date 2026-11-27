---
title: swop CLI PATH, authentication, QMD, and local DeepSeek routing
scope: machines/swop
machine: swop
status: active
confidence: high
created: 2026-09-04
updated: 2026-11-28
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

# SWOP：舊環境、服務主體和 live 驗證

- 長時間存在的 HAPI/Codex process 不讀新的 User PATH。找不到 qmd/gh 時先查 User PATH、WinGet ownership 和實際執行檔，臨時 prepend 已確認目錄或重開新 session；Node portable 路徑會隨 WinGet 版本更動。在 Windows 驗 agent-memory 的 full-Git-history validator，PowerShell 設 `PYTHONUTF8=1`，以免 CP950 在未開始驗內容前失敗。
- Git GCM OK 不代表 GitHub CLI 登入。需持久 `gh` 認證時走官方 device flow，`gh auth status`、repo API 和 Git transport 分開確認。Classic token scopes 不足會被 `gh auth login --with-token` 拒絕，臨時 CLI 授權不等同持久登入。不列印或手動搬運 GCM 憑證。
- QMD 能 lexical search 不代表 embeddings 齊全。先讀當前 config/status。Windows bash 用 `COMPUTERNAME` 取小寫 host，不能照搬 `hostname -s`；本機 QMD config/cache 在 `%LOCALAPPDATA%\Temp\qmd-memory-<host>`。集合名稱用 `memory`；新 config 忘了建 collection 會回 `Collection not found: memory`。SWOP 的 RTX 3060 已驗 GPU embed，`QMD_FORCE_CPU=1`／`QMD_EMBED_PARALLELISM=1` 只在實測 GPU 路徑失敗時作 fallback，別把例行 embed 改成 CPU。[Windows QMD 設定與 GPU 實測](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/machines/swop/qmd-windows-setup.md)
- `DeepSeek Gateway (SWOP)` 用 SYSTEM / AtStartup，監聽同機 `127.0.0.1:35001/v1`；AtLogOn interactive task 曾隨 SSH 以 0xc000013a 結束。Gateway、launcher 與 ACL 保護的 LocalMachine DPAPI 檔在 `<program-data>/DeepSeekGateway`，解密值只進 launcher Process 層。正式與 user-local exe 都核對。先 drain 連線並停原 task，只收核實 executable path 的 child，原子換檔後啟原 task。PowerShell `[IO.File]::Replace` 第三參數用明確備份路徑，`$null` 轉成空字串會報 illegal path。失敗恢復原檔和 task，不能用臨時 user process 冒充。看到 200 時另核 response upstream；見 [fleet routing](../../projects/deepseek-latch-gateway/cliproxyapi-retirement.md)。
- PowerShell 5.1 native stderr redirect 搭 `$ErrorActionPreference = 'Stop'` 會把 provider 訊息當 NativeCommandError，Gateway 可在第一次失敗輸出就退出。DPAPI/secret 前置驗證保留 Stop，僅 native invocation 改 Continue 並保留 `exit $LASTEXITCODE`。需送實際請求後查同一 listener PID，startup health 成功不足。
- 運行中的 HAPI exe 被鎖不能直接 Copy-Item 覆蓋；NTFS 可重新命名已核准舊檔，再放同名新檔。換 runner 前依 runbook 保留已知 backup、查 active session/PID，透過原 `HAPI Runner (SWOP)` Scheduled Task 恢復，確認 machine identity、Hub heartbeat 和實際 Agent canary。SSH 或 HAPI 可見性分別驗；不同 NAT 網段的歷史 LAN IP 不可直接當當前路由。
- GitHub fetch 卡住時，若 LAN SSH 可達，來源可製作以對方 commit 為 prerequisite 的增量 bundle。兩端驗 bundle，目標 source clean 才 fetch/fast-forward，再 skillshare sync。不終止別人的 pull。SSH PowerShell 讀 junction 失敗時，以同機真實 HAPI Runner principal 的 list-directory RPC 核對；source 可讀或 SSH 單獨失敗不證明技能是否於 Runner 看得到。

## WinGet portable 升級

- **登錄版本落後實際執行檔，WinGet 拒絕升級。** 用 `Get-Command` 找 live binary，查版本，將 `Get-FileHash -Algorithm SHA256` 與同版本的 `winget show -e --id Google.AntigravityCLI -v <installed-version> --source winget` 對照，再核 `Get-AuthenticodeSignature`。證明是官方套件版本漂移後才用 `winget upgrade -e --id Google.AntigravityCLI --version <target-version> --source winget --force`。用新 SSH 驗 `where agy`、版本和 WinGet 登錄。
- **安裝器說新增 alias，新 SSH 卻找不到 claude。** 先查 WinGet Packages 中官方 `claude.exe`、WinGet Links 中的入口及 User PATH。已確認缺 Links 入口時，用 cmd.exe `mklink` 對確切版本的官方 binary 恢復原入口，再在新 SSH 核對 `where claude`、`claude --version`。不用重複新增 PATH。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/swop/cli-path-auth-and-qmd.md) · [WinGet 修復的原始驗證](https://github.com/swear01/agent-memory/blob/86f38610525e57a230a7c3b87405fea76530bb1c/machines/swop/cli-path-auth-and-qmd.md)
