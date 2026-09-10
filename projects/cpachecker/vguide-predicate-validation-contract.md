---
title: VGuide predicate literal 與 validation 責任邊界
project: cpachecker
scope: projects/cpachecker
tags: [vguide, parser, smt-lib, bitvector, validation, muse]
status: active
created: 2026-08-30
updated: 2026-09-11
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


# C-syntax array 的模板路由限制（Issue #219）

`ArrayTermTranslator.hasArrayAccess` 表示存在可轉譯模板，並非只辨識 array 語法。
`collectTemplates` 只掃 `select`；現有 recognizer 要求 `bvadd` 左 operand 是 address
atom，右 operand 是可識別的 `bvshl` indexed access。若只有 constant-first offset select
和 indexed `store`，就不會產生模板，`a[9]` 轉入 scalar parser 後被報成
`variable_not_in_scope`。Prompt 明確允許 C-syntax `a[i]`，因此不能直接把此診斷算作
模型 scope 選擇錯誤；但路由問題也不證明候選語意正確或 invariant 有用。

Issue #218 四筆 first-request 原始序列、各兩個 block formulas，在 production pure-Java
helper 中均未產生 `a` 模板；六個原始拒收候選皆走 scalar route，shift-shaped positive
control 可轉譯。Root 核對 fixture hashes、JSON decoded formulas 與兩 arm 相同 class，
並重跑 helper 通過；沒有 solver/native/provider 執行。這是診斷完成，尚未修復。

擴充 constant-offset 支援前，要有 typed source/layout 或已知 indexed access 的 stride
依據；heap value width 本身不足以證明 C object layout。證據不足時保留拒收並改善能力
診斷，不能猜 stride、放寬 unknown-variable/scope checks 或宣稱修好後一定提升 solve。

# 參考謂詞必須使用 C carrier width

Issue #103 的 bakery setup 使用 BV1 宣告 `main::state_39`，在 abstraction instantiate
遇到同名符號已有其他型別的 Java `IllegalArgumentException`。原始 C 的
`SORT_1` 是 `unsigned char`，雖然註解寫「BV with 1 bits」並以 mask 限制值域；
同任務保留的 production replay log 實際宣告 `main::state_39@3` 為 BV8。
因此手寫 predmap 要依 C declaration、data model 和實際公式型別決定寬度，
不能把 masked logical domain 的位數直接當成 SMT carrier width。

例外訊息中的 `T(34)` 是不透明型別標識，不能解讀為 34-bit；recorder 的
`crash` 分類也不能直接當成 native SIGSEGV。BV1 上「等於 0 或 1」是 tautology，
改成 BV8 後才是非平凡值域切分，但仍未證明可匯入、對 refinement 有用或能解題。
保留舊 map 與失敗紀錄，修正 map 必須另行完成實際 import qualification。

來源：`reports/issue103-import-replay-qualification-20260909/outputs/` 的
`hardware-setup/cpa.log`、`hardware-replay/cpa.log` 與同一 benchmark source；
frozen code commit `b3ad20052f6c3da7d5ad321f050a4cb0d18fce0c`，修正追蹤 #239。

# 初始謂詞讀取失敗可能繼續分析

`PredicatePrecisionBootstrapper.prepareInitialPredicates()` 會捕捉 IOException 和
PredicateParsingFailedException，記錄 `Could not read predicate precision from file`
或 `Could not read predicate map` 後繼續；分析能跑到 refinement 不代表初始 map 已匯入。
驗證手寫 oracle map 時，先釘住 initialPredicates option 的 exact file/hash，檢查這些
warnings，再核對實際 retained/exported predicate 內容與型別。分開回報 parser 接受、
已觀察到的 SSA/type compatibility、後續 solver outcome；不要用 process exit 或 predmap
檔案存在代替匯入證據。此行為在 merged source `d195e94cc51f5f14f620d3a1c0c6cb906a12eb2f`
的 PredicatePrecisionBootstrapper 與 persistence/PredicateMapParser 已逐路徑核對。

# Parser 宣告成功不等於程式變數綁定正確

#241 的 ECA `Problem102_label11` 參考 map 將十一個全域 `a*` 宣告成
`main::a*`；實際 benchmark 在 file scope 宣告（lines 106–116），`main` 到
line 171565 才開始。輸出的 predmap 同時保留 `main::a*` 和 bare `a*`，證明
「有讀入公式」不能代替程式變數對應檢查；#239 報告稱這些是 main locals 的
敘述錯誤，修正追蹤 #242，舊 map/raw 保持不動。

人工 oracle qualification 要逐一核對 C declaration scope、實際 emitted symbol、
carrier type 和 SSA 版本；不能替所有符號加上 main::，也不能只比 unqualified
字串是否出現在 source。初始 SMT declarations 可產生與程式狀態分離的符號。
另須區分字面比對和內容解讀：輸出 predmap 會有 let bindings、比較式與 bit-vector
正規化，原始 assert 字串找不到並不證明未匯入；保留 checker unknown，再附具體
source/retained-formula 證據，不能為通過檢查改寫 raw 或宣稱 solver equivalence。

# 初始 precision 與首次 lookup 的觀察邊界（PR #255）

`PredicateCPA` 既有 `FINEST` 的 `Initial precision is` 透過
`PredicatePrecision.toString()` 保留 global/function/local/location-instance placement。
若舊 run 未開該 log，檔案存在或 solver timeout 不能補成已觀察到匯入。

`log.level` 控制檔案日誌，`log.consoleLevel` 控制 console，預設後者為 INFO。
只設 `log.level=FINEST` 時，`Initial precision is` 不會出現在 console capture；
要明確保留 `log.file` 指向的檔案（預設 `CPALog.txt`），並分別記錄檔案與 console
的 hash、行號及觀察結果。這在 #256 的 runtime256 產生的
`doc/ConfigurationOptions.txt` 與 `PredicateCPA.java:213` 已核對。
`log.truncateSize=10000` 是訊息字串的截斷設定，不是整個檔案大小上限；
不能從日誌總大小推論某筆 precision 是否完整，證據不足仍記 unknown。

PR #255（merge `b3ce8d90cb0513b65a3ab0bffe5febeba124168a`）在
`PredicatePrecisionAdjustment.computeAbstraction` 新增一次 INFO：
`First plain predicate-map precision lookup/handoff`。只有 bootstrap 成功解析非空的
plain predicate map 才啟用，YAML/GraphML witness 不在此觀察範圍；
`AtomicBoolean.compareAndSet` 確保共享 adjustment 時仍只記一次。

此 marker 記錄首次 lookup 的數量、位置及 precision 分類數量，不提供每個謂詞的
身份或 solver 消費證明。local/function map 的早期 lookup 可能尚未選到參考謂詞；
沒有第二筆 marker 也不能推論後續未使用。應將 parser 接受、初始 placement、
lookup、下游過濾、solver 使用與候選充分性分開，不因新增紀錄功能就更新舊實驗結論。
驗證為相關 parser placement/rejection 測試 22/22、build 和 checkstyle；未據此宣稱
四個 reference run 已完成新的 runtime qualification。
