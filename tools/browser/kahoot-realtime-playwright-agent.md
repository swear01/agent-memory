---
title: "Kahoot 即時自動作答：本機 Playwright + 快速外部 LLM API 實戰紀錄"
scope: tools/browser
status: active
updated: 2026-11-24
---

# Kahoot 即時自動作答：本機 Playwright + 快速外部 LLM API

## 架構選擇（為何不用內建 browser subagent 跑現場局）
- **內建 browser subagent**：即使改用 `page.ariaSnapshot()`（~19 ms）＋精簡 JSON 解析，每次讀題仍須經代理模型回合與截圖／HUD 產生，來回固定延遲約 **10–15 秒／題**，超過現場主持人控場的短計時題。
- **本機 Playwright + 外部 LLM API（`httpx` HTTP/2 長連線預先暖機）**：從題目出現到完成點擊／送出可壓到 **0.4–1.6 秒**。
- macOS 可直接用 `uv run --with playwright --with "httpx[http2]"` 搭配 `/Applications/Chromium.app/Contents/MacOS/Chromium` 啟動，無需額外下載 Playwright Chromium。

## Kahoot DOM 實測細節
- `https://create.kahoot.it/solo?...` 會直接重導向至頂層 `https://kahoot.it/solo?...`（非跨網域 iframe）。
- 頁面第一個 `h1` 可能是空字串；題目文字需取非空的 `[data-functional-selector="block-title"], main h1, h1, h2`。
- 選項按鈕使用 `[data-functional-selector^="answer-"]`；`[data-functional-selector="nano-next-button"]` 在題目作答期間也會存在於 DOM（帶 `aria-hidden="true"`），點擊前須檢查 `aria-hidden !== "true"` 且僅在已作答後切換。
- 現場 `/answer/result` 頁面有一個可用的 `"開啟反應選單"` 按鈕，輪詢新題目時需以 `form` 內含未停用的選項／輸入框／滑桿為準，不可只看頁面上是否有 enabled button。
- 倒數秒數與剩餘字數不出現在無障礙樹（`ariaSnapshot`）中。

## 實測可用的快速模型與 API 參數（2026-11）
- **Anthropic (`ANTHROPIC_API_KEY`)**：
  - `claude-haiku-5-5`：預設開啟 thinking 約 **1.61 s**；設 `thinking: {"type": "disabled"}` + `temperature: 0` 約 **1.18 s**（開啟 thinking 時不可傳 `temperature`）。
  - `claude-opus-5-5`：不支援 `thinking.type="disabled"`，須用 `thinking: {"type": "adaptive"}` + `output_config: {"effort": "low"}`，實測約 **1.09 s**。
  - `claude-sonnet-5-5`：關閉 thinking 須用 `thinking: {"type": "between_tools"}`，實測約 **1.57 s**。
- **Groq (`GROQ_API_KEY`)**：
  - `openai/gpt-oss-120b`（`reasoning_effort: "low"`，`response_format: {"type": "json_object"}`）：單題實測約 **0.65–0.77 s**（端到端 4 題平均 **657 ms**）。
