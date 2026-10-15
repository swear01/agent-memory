---
title: Native Claude Code 的隔離邊界與 stream 用量口徑
scope: tools/claude-code
status: active
updated: 2026-10-15
---

在 Linux 實測 Claude Code 2.1.292 的官方訂閱、`-p --output-format stream-json --include-partial-messages` 流程。以下是該版本的觀察，升版後仍須重新驗證。

隔離 HOME 與 `CLAUDE_CONFIG_DIR` 並不單獨保證只載入指定 project skills。工作目錄的祖先也可能提供 project instructions／skills；乾淨 workspace 應有自己的 git boundary，明確使用 `--setting-sources project`，並核對 debug 中的發現來源與 init registry。該版本實測未發現 `.agents/skills`，project-local `.claude/skills` symlink 指向同一份未修改內容後可發現。不要只以檔案存在就聲稱 skill 已執行，另核對真正的 `Skill` tool call；透過 Bash/Read 閱讀後續 skill procedure 與正式 Skill invocation 應分別列帳。

需要排除祖先規則檔時，實測設定 `claudeMdExcludes`，涵蓋 `**/CLAUDE.md`、`**/CLAUDE.local.md`、`**/AGENTS.md`。自訂 project skills 與 CLI inherent builtin registry 仍須分開報告；不能把 builtin metadata 說成外部自訂 skill 污染，也不能聲稱 registry 完全不存在。

同一 native CLI process 可以因 background task notification 產生多筆 `type=result`；第一個 `stop_reason=end_turn` 後 process 仍可能存活。不要看到第一筆 result 就終止程序、凍結成果或判定整個流程完成。要結合程序退出、最新 result、背景工具狀態與工作流程最終交付物判斷。

實測跨 `--resume` 的 `total_cost_usd` 與 `modelUsage` 為同一 session 的累計值；應採最後一筆，不能將每次啟動的累計值再相加。每筆 `result.usage` 則可加總，包含同一程序內後續 background notification 的 result。加總後與最後的 `modelUsage` 各類 token 核對。`output_tokens` 已包含 `output_tokens_details.thinking_tokens`，thinking 可另列明細，不能再加回 output。cache read、cache creation 與未快取 input 分別列出。訂閱登入仍可回傳 list-price API-equivalent estimate；它不是實際訂閱扣款。

背景等待工具不要使用會匹配自身 command line 的 `while pgrep -f <script>; ...`。曾確認 synthesis 已退出，但 waiter 的 shell 自己仍含 script 名稱而永遠等待。應追蹤已知 child PID／job handle；若修復既有卡住的 waiter，先驗證完成日誌與精確 parent/cgroup，只終止已確認的監視 shell，保留原生 agent 與工具產物。
