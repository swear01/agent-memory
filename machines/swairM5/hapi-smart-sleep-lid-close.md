---
title: SwairM5 合蓋防睡眠：HAPI smart-sleep 與 Claude 活動偵測
scope: machines/swairM5
machine: SwairM5
tags: [pmset, lid-close, clamshell, hapi, claude-code, launchd, betterdisplay, lid-display-switch]
status: active
created: 2026-10-25
updated: 2026-11-22
---

用 `pmset -g` 檢查 `SleepDisabled 1`。Claude Desktop 的 keep-awake、`caffeinate` 和 Amphetamine 類 assertion 擋 idle sleep，在電池合蓋仍可能出現 `Clamshell Sleep`。BetterDisplay 與 `xyz.hapi.auto-virtual-display` 負責顯示器切換。

## 自動防休眠

- `~/Library/Application Support/HAPI/` 的 LaunchAgent `xyz.hapi.smart-sleep-watcher`（`hapi_smart_sleep.py`）每 2 秒更新 `smart-sleep.state`。ActivityLatch 在 busy 開啟，idle 10 秒或 unknown 60 秒後解除。常駐 Claude 進程不能表示工作中。CLI、桌面 Code 與 HAPI Claude 的 transcript `~/.claude/projects/*/*.jsonl` 在 180 秒內更新算 busy。Cowork 和一般雲端聊天無此訊號。長時間無 transcript 更新可能睡著，電池放包包則留意工作持續時發熱。
- root LaunchDaemon `xyz.hapi.smart-sleep-helper`（`/Library/Application Support/HAPI/hapi-smart-sleep-helper.sh`）把超過 20 秒的 state 視為過期，退出亦重設 `disablesleep 0`。直接手動設值可能幾秒後被校正。先查 state 新鮮度、launchd、`/var/log/hapi-smart-sleep.log`。修改 watcher 用 `launchctl kickstart -k gui/$(id -u)/xyz.hapi.smart-sleep-watcher` 及同目錄 `python3 -m unittest test_hapi_smart_sleep`；原始備份 `backups/*.20261025`。Watcher 已實測 busy 時為 1，idle 回 0 有單元測試覆蓋。

## 手動倒數

用 `~/.local/bin/keep-awake 7200`，正整數秒數，與「合蓋不休眠」捷徑共用互斥鎖和心跳，已有手動倒數會拒絕。用 `pmset -g`／`IOPMrootDomain` 讀回 1/Yes 及到期的 0/No。已驗證短倒數及兩小時倒數啟動。實體合蓋與 watcher 併行未實測。若 CLI 不可用且 watcher 確定未載入，歷史驗證用使用者 launchd 限時刷新既有值為 1 的過期 state。保存原 mtime，到期還原並移除臨時工作，讓 helper 過期復原。避免兩個 writer 共寫 state。

## 虛擬螢幕切換加固

- 記憶體壓力、BetterDisplay 無回應和重試重疊曾伴隨 WindowServer 溢位與 `ap_panic`。
- **目前行為（`~/Library/Application Support/HAPI/lid-display-switch.swift`）**：
  - 狀態檢查優先使用原生 `CGGetOnlineDisplayList` + `NSScreen.screens`（`detectVirtualDisplayNative`）；開蓋無虛擬螢幕或合蓋已掛上虛擬螢幕時 **0 次 BetterDisplay CLI 呼叫**，僅在真正切換或鏡像／未知狀態時才回退 CLI。
  - 開蓋設回內建主螢幕改用原生 `CGConfigureDisplayOrigin(..., 0, 0)` + `CGCompleteDisplayConfiguration`（必要時先解除鏡像），不依賴 BetterDisplay。
  - CLI timeout 縮短為 5 秒；連續超時 2 次自動 `SIGTERM → SIGKILL` 並 `open -g -b pro.betterdisplay.BetterDisplay` 背景重啟（冷卻 120 秒）；連續超時達 4 次進入 300 秒熔斷退避。
  - 嚴格單一執行排程（`isReconcileRunning` + 清除 `weakWork`）：`health check` 與失敗期間的 `display reconfiguration` 不再重置退避，僅 `lid change` / `screens wake` / `session active` 會立即搶佔。
- **部署與驗證**：依 `macos-launchagent-ad-hoc-binary-deploy.md` 以新 inode 替換並執行 `launchctl bootout` + `bootstrap`。已通過 `xcrun swiftc`、`--self-test`、`codesign --verify` 與啟動零 CLI 呼叫驗證；尚未實測實體合蓋／開蓋與 BetterDisplay 實機卡死復原。舊版備份於 `~/Library/Application Support/HAPI/backups/lid-display-switch-20261122-pre-plan-b/`；`transfer_MAC` snapshot 尚未同步。

[舊版診斷與逐次驗證](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/swairM5/hapi-smart-sleep-lid-close.md)
