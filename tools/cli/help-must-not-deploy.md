---
title: "部署器查詢 help 不應觸發部署"
scope: tools/cli
status: active
updated: 2026-09-16
evidence_digest: 30c690153c08ab80471b7da0c7f16213e19c9b517a4fac84a2cd5e5b8ec1413a
---

# 部署器查詢 help 不應觸發部署

助手為核對部署器參數而執行腳本，該腳本未處理 help 且對任何參數照常部署，導致版本再次增加。

有副作用的 CLI 要在開始修改前處理 help 與拒絕未知參數；用無副作用測試確認查詢參數不會改版本或產物。

這是歷史事故自述，沒有 help 分支修復證據。接受意外增加的版本是當時的收尾選擇，不是一般復原規則。

子命令退役也要跑解析路徑。移除 `geminiCommand` 後，舊 `hapi gemini` 曾由 `resolveCommand` 當成預設 Claude 啟動並把 `gemini` 當參數。透過明確退役錯誤或共用的 unknown-command 分支避免 fallback；原始記錄只有 tombstone 已提交，沒有執行驗證或 merge 證據。見 `git show 2505b5e:tools/hapi/removed-subcommand-default-fallback.md`。
