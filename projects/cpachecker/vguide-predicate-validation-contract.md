---
title: VGuide predicate literal 與 validation 責任邊界
project: cpachecker
tags: [vguide, parser, smt-lib, bitvector, validation, muse]
status: active
created: 2026-08-30
updated: 2026-08-30
---

# 先區分 generation 與 parser representability

Issue #152 的 Muse final content 使用標準 SMT-LIB hexadecimal bit-vector literal，
例如 `#x00000064`。原本 `VocabularyGuide.parseBvExpr` 沒有解析 `#x`，而是把整個 token
當成變數，導致 scope validation 以 unknown variable 拒絕。這是 consumer/parser bug，
不能據此判定 prompt 或模型沒有找到 predicate。

Issue #163 / PR #164 在共用 parser boundary 修正並合併：

- `#bBITS` 的寬度是 binary digits 數；
- `#xHEX` 的寬度是 hexadecimal digits 數乘 4；
- 既有 `(_ bvK W)` 支援保持不變；
- 非標準 `(bv K W)` 仍回 parse error，不得靜默重寫或修補 model output。

Base 是 `4da075b68273228788397fc7f167f32e292960ec`，修正 head 是
`49e2f24ea11a0c34d28e5f21509c91de35bacb64`，merge commit 是
`be2d2fffa5e44d55361b8e745ba464feefd778a7`。

# Artifact audit 與判讀

- 既有 final-content artifact 有 4 個檔案、共 21 個有效 `#x` tokens。
- final content 沒找到 `#b`；binary 是同一 grammar branch 的 latent sibling bug，因此一併
  以回歸測試覆蓋。
- 一個 Muse final-content artifact 有六個 `(bv K W)` candidates；這是 model-side malformed
  syntax，修 parser 時不可把它納入接受語言。

回報實驗時分開記 generation、validation、trajectory、verdict、cost。Parser 修正只改變
validation representability；沒有新正式 replay/run 時，不得產生新的 trajectory、verdict、
population、timing 或 cost claim。

# Regression evidence

- RED：pipeline 18 tests 中新案例如預期失敗，兩個標準 literals 都是 0/2 validated。
- GREEN：`VocabularyGuideTest` + `PredicateValidationPipelineTest` 29/29 通過，並確認
  `(bv K W)` 仍以 parse error 拒絕。
- Full unit suite：4321 tests、0 failures/errors；configuration checks：3880、0
  failures/errors。
- Canonical `ant all-checks` 與 exact base 都只停在相同 78 個 inherited
  `forbiddenapis` findings；此 diff 沒新增 finding。

