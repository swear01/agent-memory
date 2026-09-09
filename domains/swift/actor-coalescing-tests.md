---
title: Swift actor single-flight 測試要控制真正的 suspension boundary
scope: domains/swift
status: verified
updated: 2026-09-09
---

Swift actor 的兩次 state read、`Task.yield()` 或同優先序工作，不能證明另一個 `Task` 已經加入 in-flight 操作。依賴 FIFO 排程的 coalescing 測試會偶發發出第二次請求，即使 production single-flight 實作正確。

測試可用接受 `isolated MessageWindowController` 的區域 async 函式：先建立 response-release `Task`，明確引用該 isolated actor（例如讀取 `controller.state`），再直接 `await controller.syncTail()`。release task 繼承同一 actor isolation，必須等目前呼叫加入 in-flight task 並 suspend 才能執行。保留精確的一次請求斷言；多緩衝一個回應可讓失去 coalescing 的實作明確失敗而不掛住。

不要改成 sleep、重試 CI、放寬請求數，或為測試增加 production callback。

驗證：HAPI PR #1771，Swift 6.1.2 Linux，目標測試重複 100 次通過；暫時略過 joining guard 時測試偵測到 2 次請求；還原 production source 後 509 個 package tests 全過。macOS 行為另由 CI 驗證。

Swift 原始碼文件的 `@_inheritActorContext` 節說明：instance actor context 必須透過明確捕捉 isolated parameter 才能繼承。此處使用標準 `Task` 的內建行為，無需自行標註 underscored attribute。
