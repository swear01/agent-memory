---
title: "驗證下游失敗時輸入必須先通過上游階段"
scope: tools/testing
status: active
updated: 2026-09-16
evidence_digest: 929b1e3f6e8273f9cbfc8b7f5f3a00f6869ca160bd1ac206c8d9e7ce7831b10a
---

# 驗證下游失敗時輸入必須先通過上游階段

使用者指出提供的 Mermaid 範例修復前也不會 crash；助手改找 parse 通過、render 才報錯的 gantt 範例，以接近真正修復路徑。

讓測試輸入到達受修正階段，再比較舊版與修正版的可觀察結果。上游 parse 已拒絕的案例不能證明 render-error handling。來源僅提出改良手測與預期，沒有使用者實測或完整前後驗證；特定例子也受 Mermaid 版本影響。

Mermaid 的測試要分開驗原始碼 fallback 與渲染器留下的 `Syntax error in text` SVG；後者才是 `suppressErrorRendering` 的比對目標。聊天正文可能包含相同錯誤文字，不能在整頁做唯一判定。歷史比較未提供部署驗證。見 `git show 9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e:tools/hapi/mermaid-error-artifact-validation.md`。
