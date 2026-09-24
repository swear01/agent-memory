---
title: AIsimpV direct-API RTL pilots and output budget
scope: project
project: rtl-relate
status: active
updated: 2026-09-24
---

# Direct-API skid8 pilots

專案 PR #24 的 `docs/reports/direct_api_pilots.md` 保存 Muse Spark 1.3 Contributor 與 DeepSeek V4.1 Flash 的正式結果。兩者使用同一凍結 skid8 輸入與獨立形式驗證；模型沒有 gold certificate 或工具。固定 C/A 證書任務都在第一個候選取得 `ACCEPTED`／`SAFE`（DeepSeek 62.213 秒、Muse 88.070 秒）。

Joint RTL 任務原設每模型四次、900 秒，兩者都採 high reasoning 和 16,384-token output cap。Muse 第 1 次截斷、第 2 次 stale abstract hash，第 3 次得到 18→10-bit、`ACCEPTED`／`SAFE` 候選，累計 304.273 秒。DeepSeek 四次全部把 16,384 tokens 花在 reasoning，`finish_reason=length`，沒有可檢查候選，累計 305.337 秒。這是輸出預算失敗，不能記為形式驗證反例。

另立的 DeepSeek 65,536-token 補充輪，第一個 run 在 115.075 秒收到 gateway HTTP 503；使用相同凍結設定的獨立重試，在第 2 次得到 10-bit、`ACCEPTED`／`SAFE` 候選，重試 run 累計 150.586 秒。兩份保存的 Muse／DeepSeek joint 候選各自又在新目錄重驗通過。PR #24 將 DeepSeek 預設 `max_tokens` 提高到 65,536；Muse 保持 16,384。16K 原始四次與 64K 的 503 均保留，不能把補充輪合併成原 protocol 的第五次嘗試。

兩份 joint 候選仍是已知的 hidden-data cutpoint 類型，且只有 skidbuffer 一個家族；數百秒搜尋時間不支持端到端 proof 加速或新的抽象方法。gateway 成功回應只證明本輪請求可走 `opencode-go-1`，不證明舊 403 failover 缺口已修。官方 DeepSeek API 的 402 仍與本機 gateway 路徑無關。
