---
title: VGuide predicate literal 與 validation 責任邊界
project: cpachecker
scope: projects/cpachecker
tags: [vguide, parser, smt-lib, bitvector, validation, muse]
status: active
created: 2026-08-30
updated: 2026-09-21
---

## 先分清字串、C 語義與真正綁定

- **`#x00000064` 被當 unknown variable。** 舊 parser 缺標準 literals。已合併 #164 支援 `#xHEX` 每位 4 bits 與 `#bBITS` 每位 1 bit，`(_ bvK W)` 保留；`(bv K W)` 仍是 malformed。驗 parser representation 和 scope，再看 trajectory 或 verdict。
- **`a[9]` 走 scalar parser 卻報 `variable_not_in_scope`。** 舊 `ArrayTermTranslator.hasArrayAccess` 表示已有可轉譯模板；#219 的常數在前 offset select 加 indexed store 走不到僅識別特定 `select` 形狀的模板。先核 C layout、stride 與 active-function context；不用 heap value width 猜 stride。四筆原始 request 和 positive control 重現，該診斷不等於當時已修。
- **Predmap 能解析，但 operand 綁錯。** 用原 C 宣告、machine model、實際 emit symbol 和 SSA 核對。例如 `unsigned char` 的 BV8 carrier 不因註解或 mask 的邏輯域而改 BV1，file scope 全域也不加 `main::`。看保留公式的結構，let 和 BV 正規化會使原 assert 字串找不到；保留 unknown 而不改 raw artifact。
- **Native shadowing 或 legacy scope 出錯。** PR272 用 lexical DECL map，不能靠放寬歧義 guard；其固定 zero_sum4 對照實際 precision 34→36，但各兩次皆 timeout。後續 PR274–277 legacy scalar 使用 head function vocabulary，array/native 用最後一次 aligned PathFormula，缺 context 時不得回頭採早期 SSA。共用 parser 要保留 n-ary operands 與 BV width；用真正 BV solver（本案 MathSAT5）驗 modular 行為。

## 從匯入到求解

- **啟動成功但 initialPredicates 沒生效。** 鎖 option 指定的檔案及 hash，檢查 `Could not read predicate precision from file`／`Could not read predicate map`；bootstrap 可記警告後繼續。開 `log.level=FINEST` 並保留 `log.file`（預設 CPALog.txt），console 是另一個 `log.consoleLevel`；`log.truncateSize` 是單訊息限制。核對 `Initial precision is` 的 global／function／local／location-instance 分佈。
- **只看到 first lookup／`passed` 就說 solver 用到了。** `First plain predicate-map precision lookup/handoff` 僅非空 plain map（不含 witness），記第一次位置及計數，不提供身份。#254 的 `passed` 在 `newProverEnvironment`／`push(f)` 之前；同 run 可依內部 identity 連接，跨 run 用 formula、head、disposition。分開報 parser、實際 inject／首次匹配、solver operation、verdict 和有配對控制的 usefulness。
- **空 JSON、截斷或部分 injection 混成成功。** Jackson 解析 schema，v1 空 candidates 的 `parse_ok=true`、`empty_candidates` 與 blank／malformed／wrong schema 分類。用完整 head/formula equality 去重並只記已寫到 precision 的 bindings，部分失敗不能以預計成功數掩蓋。PR274–277 有 110 項 focused tests 與舊 167 筆回應相容核對；無重跑 trajectories 或新增 solve 證據。

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/cpachecker/vguide-predicate-validation-contract.md)
