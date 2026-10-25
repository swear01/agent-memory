---
title: Claude Code 模型清單：main/overflow 分區與 initialize 回應
scope: tools/claude-code
tags: [model, model-picker, initialize, stream-json, hapi, 1m, context-window, get_context_usage]
status: active
updated: 2026-10-25
---

## 事實（Claude Code 2.1.302 驗證）

- 伺服器下發的 model catalog 每列有 `section: "main" | "overflow" | "deprecated"`。本機快取在 `<home>/.claude/cache/model-catalog/*.json`（`catalog.models[].section`），屬內部格式，不要讓產品程式依賴它。
- TUI `/model` 不會隱藏舊模型：丟掉 `deprecated`，`main` 排前、`overflow` 排後，一次顯示 10 列，其餘捲動（「… +3 models」只是捲動提示，不是隱藏）。claude.ai／desktop 的 picker 才會把 overflow 收進「More models」。
- `--input-format stream-json` 送 `{"type":"control_request","request_id":"x","request":{"subtype":"initialize"}}`，回應的 `response.response.models[]` 只有 `value/resolvedModel/displayName/description/supportsEffort/supportedEffortLevels/...`，**沒有 `section`**。
- 當時實測：main 4 列剛好是 alias（`opus`→Opus 5.5、`fable`→Fable 5.1、`sonnet`→Sonnet 5.5、`haiku`→Haiku 5.5），overflow 8 列都是完整 id（Haiku 4.5、Sonnet 5、Opus 5、Fable 5、Opus 4.8/4.7/4.6、Sonnet 4.6）。但曾出現 main 列是完整 id 的情況（alias 尚未切換時的 `claude-fable-5-1`），所以不能只用 `claude-` 前綴判斷。
- 可靠的推導：依 CLI 順序，某列的 family（`claude-opus-4-7`→`opus`）在前面已出現過，就是 overflow。
- `[1m]` 預設：opus/sonnet/fable/haiku alias 現在都解析到原生 1M 模型，官方文件說 `opus[1m]`/`sonnet[1m]` 對它們「no effect」，`/model` 也沒有 1M 列；只有 Opus 4.6 / Sonnet 4.6 還需要 `[1m]`。

## 取得實際 context window（2.1.302 驗證）

- `system/init`、`initialize`、assistant `usage` 都沒有 context window 欄位。local mode 的 transcript 也沒有。
- `result.modelUsage[model].contextWindow` 是原始視窗，每回合結束才有。它會反映 `CLAUDE_CODE_DISABLE_1M_CONTEXT`（1,000,000 → 200,000），但不會套用 `autoCompactWindow`。
- stream-json control request `{"subtype":"get_context_usage"}` 回 `response.response.{model,maxTokens,rawMaxTokens,autocompactSource,autoCompactThreshold}`。`maxTokens` 是有效視窗，也就是 `/context` 顯示的值，會套用 DISABLE_1M 和 settings `modelSettings.<model>.autoCompactWindow`（`rawMaxTokens` 也一樣被 cap）。它在回合進行中也能呼叫，回應比第一則 assistant 早。
- 注意：probe 時 prompt iterable 不能結束，否則 stdin 被關，request 永遠等不到回應。
- HAPI #2153（e5c6f0f5a）：remote mode 每次 init 都呼叫一次 `Query.getContextUsage()`，結果優先於 result 的原始值。local mode 沒有 control channel，只能用 env／settings 推斷，可以併進 #2088。

## HAPI 對應

- upstream PR #1948（junmo-kim）用 initialize 探測取得模型清單；fork 在 v0.30.7.6 carry。
- 2026-10-25 開上游 PR #2153：從 shared／Android／iOS 拿掉 Sonnet/Opus/Fable 1M。context 預算 fallback 和 CLI 第一回合 seed 都把 alias 與任何 `[1m]` 視為 1M，因為 local mode 永遠走 fallback。
- 「收合舊模型」只有 carry 了 #1948 才會出現，上游 main 沒有這個問題，所以不另開上游 PR（#2154 已關閉）。修正分兩處：fork overlay commit `72604bcae` 等下次 release 時以 `regenerate overlay -` 接在 #1948 carry 後面；另外在 #1948 留言建議作者採用（swear01/hapi@33e3003eb）。
- 2026-10-25 使用者決定：#2153 改成只提供四個 alias（Opus、Fable、Sonnet、Haiku，順序照 `/model`），下次 fork release **不再 carry #1948**，`72604bcae` overlay 也一起丟。拿掉 #1948 會失去固定舊版本和依模型縮減 effort，但四個 alias 都支援全部五級。rehearsal 時要注意後面 carry 中改過 #1948 檔案的 patch 可能要重新解衝突。
