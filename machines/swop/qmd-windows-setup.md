---
title: Windows 上跑 QMD（swop / Swear01_PC）的設定差異
scope: machine
machine: swop
status: active
created: 2026-09-21
updated: 2026-09-21
tags: [qmd, windows, windows-paths]
---

# Windows 上的 QMD 設定（swop / Swear01_PC）

shared-memory skill 的 QMD 範例是 Linux 路徑，在 Windows 上要換：

- `$(hostname -s)` 在 Windows bash 會報 `unknown option -- s` → 直接用 `%COMPUTERNAME%`（本機 = `Swear01_PC`）。
- QMD config/cache 放在 `%LOCALAPPDATA%\Temp\qmd-memory-swear01-pc\{config\qmd,cache}`。
- collection 名稱用 `memory`（`qmd collection add <path>` 預設會取名成路徑，要 `qmd collection rename <path> memory`）。

## GPU embed（踩過的坑）

- **失敗案例（2026-09-21）**：把 skill 文件裡的 `QMD_FORCE_CPU=1 QMD_EMBED_PARALLELISM=1` 當常規參數套上，556 檔 embed 在 CPU 上跑幾分鐘仍未完成。那是「VRAM 不足報錯時」的 fallback，不是預設。
- 本機 RTX 3060 12GB（idle 時只佔 ~2GB）：**預設不加 FORCE_CPU**。GPU embed 556 檔 / 893 chunks = 1m 08s；CPU 慢一個數量級。增量 embed（幾個新檔）= 幾秒。
- 規則：先 `nvidia-smi --query-gpu=memory.used --format=csv` 確認 VRAM，只有真的不足/QMD 報錯才加 `QMD_FORCE_CPU=1` 重試。
- 首次使用先 `qmd collection add`（index.yml 預設 `collections: {}` 會報 `Collection not found: memory`），再 `qmd update` + `qmd embed`。