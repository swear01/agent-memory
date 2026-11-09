---
title: 從 Claude Code 用 codex exec 問 ChatGPT（唯讀諮詢）
scope: tools/codex
tool: codex
status: active
updated: 2026-11-09
---

# 從 Claude Code 用 codex exec 問 ChatGPT（唯讀諮詢）

搜尋關鍵字：問 ChatGPT、問 GPT、consult GPT、second opinion、codex exec read-only、gpt-6.1-sol。

使用者說「問 ChatGPT」時，指的是用 codex CLI 背景唯讀諮詢（2026-11-08 至 11-09 的 CVSD HW3 共 9 輪都這樣做）：

```bash
S=<scratch dir>
cat > $S/ask.md <<'EOF'
<題目、目標數字、已量測結果、已排除清單、要讀的檔案絕對路徑、要求的輸出格式與字數上限>
EOF
(codex exec -m gpt-6.1-sol -s read-only -c model_reasoning_effort=high \
   -o $S/answer.md - < $S/ask.md > $S/log.txt 2>&1 &)
```

- prompt 從 stdin 讀（`-` + `< ask.md`），所以不會卡在 `Reading additional input from stdin...`
  （那個問題見 [[exec-sites-deploy-from-claude]]）。`-o` 只寫最後答案；log 有過程。
- `-s read-only`：它能讀 repo 與指定檔案、能上網查（例如看 Guacamole 原始碼），但不會改檔。
- high effort 大約 5–25 分鐘。等的時候做別的事，不要輪詢。
- 一定要把**已量測的數字和已排除的方向**放進 prompt。缺這些時它會重提已失敗的方案；有這些時它會直說「沒有可達成目標的方案」，不硬湊。
- 運維問題也適用（2026-11-09：Guacamole 連線上限的即時解法，medium effort 約 3 分鐘，答案附原始碼連結）。
  它給的做法仍要實測：它說一般使用者可逐筆 DELETE `activeConnections`，但 ADFP 站上列表永遠是 0 筆，
  實際可用的是登出 token 再 `adfp-vpn login`（見 [[adfp-vpn]]）。
- 同一題要第二意見時，可另開 Claude Fable 的 Agent（`model: fable`, `effort: high`），只讓它出 idea、不實作；
  要追問時用 SendMessage 接續同一個 agent，它保留已讀內容，比重開便宜。
