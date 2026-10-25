---
title: Claude Code 模型清單：main/overflow 分區與 initialize 回應
scope: tools/claude-code
tags: [model, model-picker, initialize, stream-json, hapi, 1m]
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

## HAPI 對應

- upstream PR #1948（junmo-kim）用 initialize 探測取得模型清單；fork 在 v0.30.7.6 carry。
- 2026-10-25 開 PR #2153（從 shared／Android／iOS 拿掉 Sonnet/Opus/Fable 1M；context 預算 fallback 把 alias 與任何 `[1m]` 視為 1M，因為 local mode 永遠走 fallback）與 PR #2154（疊在 #1948 上：用 family 規則標 `older`，composer 用「Older models」toggle 收合、New Session 用 optgroup）。兩者後合併的一方要調整 #1948 在 `modelOptions.test.ts` 的 `opus[1m]` legacy preset 測試。
- 已知殘留：CLI `sdkToLogConverter` 第一回合 context window 對裸 alias 仍 seed 200k，要等 result 才修正。
