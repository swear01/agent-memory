---
title: Unity Pipeline run_script/eval 不適合長時間計算
scope: tools/unity
status: active
updated: 2026-10-31
tags: [unity, pipeline, run_script, eval, domain-reload, dotnet]
---

- `eval` 主執行緒工作 5 s 就回 timeout，但工作仍在 Editor 裡跑完（會寫出檔案），期間 Editor 卡住。
- `run_script` 回傳 `Task<string>` 並把計算丟到 `Task.Run` 也沒用：Pipeline 在主執行緒等待 Task，`console` 等命令照樣 timeout。
- 背景執行緒還在跑時觸發 recompile，domain reload 會等該執行緒結束，Editor 長時間卡住；只能 kill 自己的 Editor（先確認 LicensingClient 的 parent 不是它）再重開。
- 作法：純 C# 模擬寫成 dotnet console 專案（`<Compile Include>` 引入 runtime 檔案），Editor 只跑測試與 live 檢查。
