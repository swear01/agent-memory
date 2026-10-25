---
title: Claude remote permission modes must not launch natively in emulated modes
scope: tools/hapi
project: hapi
tool: claude
status: active
confidence: high
tags:
  - hapi
  - claude
  - permissions
  - yolo
  - bypassPermissions
source_refs:
  - tiann/hapi#2158
created: 2026-10-25
updated: 2026-10-25
---

# Claude remote：模擬型 permission mode 不能用 native 啟動

## 症狀

Remote Claude session 用 Yolo（`bypassPermissions`）開，之後在 web 切回 Default，
UI 顯示 Default，但 Bash 和檔案修改還是不經詢問直接執行。`acceptEdits` → Default
也一樣，只是範圍限於 edit。

## 根因

- `claudeRemote` 用第一個 turn 的 mode 當 native `--permission-mode` 啟動 claude。
- `runClaude.ts` 的 mode-gate hash 只對 `plan` / `auto` 重啟 process；default、
  acceptEdits、bypassPermissions 之間切換不重啟，也從不送 `set_permission_mode`。
- 設計上 acceptEdits / bypassPermissions 是在 `canCallTool`（`PermissionHandler`
  讀即時 session mode，#735）模擬。但 claude 以 native bypass 跑時根本不呼叫
  `canCallTool`，HAPI 的 Default 檢查完全失效。
- 會中招的入口：web 建立時選 Yolo（#1751）、resume Yolo session（#460）、
  `hapi claude --yolo` 交接到 remote、Default 切 Yolo 後因換 model/effort、
  `/clear`、`/compact`、切 plan/auto 而重啟。要等下一次重啟才會恢復。

## 修法（PR tiann/hapi#2158）

acceptEdits / bypassPermissions 一律以 native `default` 啟動，讓每個工具呼叫都經
`canCallTool`；plan / auto 仍 native 傳遞。2026-10-25 在 isolated hub + Claude Code
2.1.302（haiku）端到端驗證：修前切回 Default 後 `touch b` 直接執行；修後出現
pending `Bash` permission request。

## 已驗證的 Claude Code 2.1.302 行為

- `--permission-mode bypassPermissions` 與 `--dangerously-skip-permissions` 等價：
  兩者都能用 control request `set_permission_mode` 切到 default 再切回 bypass。
- 用 `--permission-mode default` 啟動的 process 不能升級到 bypass：
  `Cannot set permission mode to bypassPermissions because the session was not launched with --dangerously-skip-permissions`。
- native bypass 下 Bash 不送 `can_use_tool`，但 `AskUserQuestion` 仍然會送，
  所以 Yolo 下 Claude 仍會問問題。
- Remote 模式 `filterCatalogAffectingClaudeArgs` 會丟掉 `--dangerously-skip-permissions`，
  所以 web Yolo 與 CLI `--yolo` 在 remote 啟動參數完全相同；差異只在 local TUI。

## 尚未修的相鄰問題

Local（TUI）模式 `claudeLocal.ts` 不把 permission mode 轉成 CLI flag：
`hapi claude --permission-mode X` 與 terminal `hapi resume` 會以 default 開 TUI，
hook 再把 HAPI 狀態覆寫成 default（#460 引入）。`--yolo` 不受影響。
