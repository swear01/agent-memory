---
title: CPAchecker VGuide 實驗 provenance 與 preregistration 陷阱
project: swear01/cpachecker
scope: project
tags: [vguide, experiments, provenance, preregistration]
status: active
created: 2026-08-30
updated: 2026-09-08
---

# Frozen benchmark pairing

`docs/vguided-cegar/benchmark_sets/loops_reachsafety_unreach.list` 的 SHA-256 `c072daf0...6ae` 與 SV-Benchmarks commit `2e1723fde6aa65a250dcb677efa45edaa4b6b631` 是同一組 frozen input。不要因較新的 checkout 是 clean 就混用；`9cf91981...d` 缺少其中 26 組 source/task 路徑。Formal run 前要同時驗證 commit、clean tree、manifest hash、764 組 source/task 全數存在。

# Issue #166 G0 邊界

完整 764-task source-only census 的 deterministic diagnostic 是 6 個 structural matches，其中 5 個已在 exposure denylist，只有 `c/loop-invgen/heapsort.yml` 這 1 個 unexposed candidate。預先規定的 continuation threshold 是至少 24 個、涵蓋至少 3 個 strata，因此 #166 在 G0 停止，沒有 Stock gate、Muse generation 或 consumer replay。這不能推論 prompt 泛化成功或失敗；#149 的已接受單案例結果也不受影響。

該 census 的 GitHub preregistration comment 晚於 run 完成一秒，正式證據標為 procedural-invalid，只保留 deterministic diagnostic。不要用看過結果後的重跑假裝恢復 blind preregistration。

# 安全發布 preregistration comment

GitHub comment 若含 Markdown backticks、`$()` 或 launch command，不得插值進 shell command。Backticks 會被 shell 當成 command substitution，可能在 comment 建立前直接啟動實驗。先把 body 以正常檔案編輯流程建立，再執行：

```bash
gh issue comment ISSUE --repo OWNER/REPO --body-file COMMENT.md
```

Launch 前再讀回遠端 comment，確認 body hash、內容與 timestamp；remote comment 尚未存在就不得啟動 formal job。

# Exact request hash 與 replay cache identity

`LlmResponseCache` 的 key 是 production Java 產生之完整 request JSON bytes 的 SHA-256，不是
只有 prompt hash。Request body 同時包含 exact system/user messages、provider、model、
`max_completion_tokens`、stream flags、provider-specific response schema，以及
thinking/reasoning settings；任一欄位或 JSON schema 改變都應該 fail-closed cache miss。這個
hash 設計適合 exact replay，不應改成忽略 provider/model 的寬鬆 key。

Issue #217 證實：舊 runtime 的八筆 recorded hash equality 只能證明輸入 provenance；
新 runtime 要靠新 production request 的 hash-addressed replay hit 與實際 prompt bytes
驗證。Cache namespace 也要明確對齊：預設 `default` 不會命中既有 case namespace。
修正設定時保留失敗 attempt，另建 attempt 目錄；不要覆寫原 log 或漏算次數。
若 first-request 已完成、之後 solver timeout，可接受 request qualification，但整題仍為
incomplete，不能寫成 verifier PASS。

# Dump 的 commit 是工作目錄資訊，不是 runtime 身分

`VGuideAnalysisDumper.readGitCommit()` 在 process cwd 執行 `git rev-parse HEAD`。
Issue #217 即使透過正確的 `PATH_TO_CPACHECKER` 選到新 arm，從 navigation worktree
啟動仍會把 navigation HEAD 寫入 `run_manifest.json`。實際 startup version 與 classes/
libraries hashes 已另行確認新 runtime；不得為了對齊而修改既有 raw dump。
之後每個 arm invocation 必須同時指定 cwd 為該 arm worktree、明確 launcher/runtime
與 JDK，再核對 dump commit、startup version 和 runtime manifest。新增環境變數無法
修正這個 cwd-derived 欄位。

# 明確固定 replay provider 與序列

Current main 預設 `VGUIDE_LLM_PROVIDER=meta` 與
`VGUIDE_LLM_MODEL=muse-spark-1.2-contributor`。Java client 不讀 retired
`DEEPSEEK_MODEL`；historical DeepSeek replay 必須明確設定 `VGUIDE_LLM_PROVIDER=deepseek` 與
`VGUIDE_LLM_MODEL`。不要跨 runtime commit 抄舊 request hash。

也不要假設同一 task 的不同 arm 共用整條 hash sequence：第一個 response 是否注入 predicate
會改變後續 refinement、prompt 與第二個 request hash。每個 arm 應先以 production Java client、
明確 current request identity、local deterministic oracle 與 `VGUIDE_LLM_RECORD_DIR` 建立自己的
cache；oracle ledger 的 raw-request SHA 必須等於 recorded/dump request hash。Freeze cache、
prompt/request sequence 與 verifier 後，formal stage 只用 `VGUIDE_LLM_REPLAY_DIR`。禁止在 formal
cache miss 後手補 path 再把同一 attempt 重跑。

# Fail-closed 只停止不可信 attempt，不停止研究工作

Replay cache miss、schema mismatch 或 provenance mismatch 必須讓當下的 consumer run
fail-closed，避免錯誤 response 被當成 evidence；但這只是 integrity boundary，**不是 agent 的
工作終止條件**。自造成且可修復的 harness/config/cache 錯誤不得回報成整個研究 blocked。

Agent 在 fail-closed 後必須自行完成 recovery ladder：

1. 保存 failed attempt、expected/actual hash、effective provider/model/options、log 與 artifacts，
   並在 tracking issue 標記該 attempt invalid。
2. 用 production Java request path 調查 exact body drift；先查 effective env/config，再用 local
   deterministic oracle + `VGUIDE_LLM_RECORD_DIR` 捕捉實際 request。不得猜 hash，也不得改成
   prompt-only key。
3. 若尚未 freeze，修正 harness、為各 response arm 重建 cache、跑 ledger-SHA equality 與 pure
   replay qualification，直到 preflight 通過。
4. 若已 freeze，保留原 attempt 不動；在 freeze 外完成同樣修復，將新的 cache/request/response
   hashes 與原因更新到 issue，preregister 新 attempt 後繼續。
5. 只有缺少必要權限/credential、需要會改變研究問題的使用者決策，或同一不可控外部 blocker
   經至少三次有記錄的 recovery 仍重現時，才停止整個任務並請求介入。

禁止的兩個極端都是錯的：不能在 cache miss 後 silent live fallback，也不能因為 formal attempt
必須 stop 就讓 agent 停止調查與後續合規 retry。Issue 的 stop rule 要明確分開
`attempt_stop`、`recovery_action` 與 `task_stop`。

# Pool host admission 與 NFS 平行啟動

正式 CPU-isolated pool wrapper 不可把 `local` 當成方便的第四分支，除非先以 `hostname -s`
斷言它就是 frozen allowlist 中的主機。Issue #166 fixed-three generation attempt 001 曾因 wrapper
接受本機 `mazu` 而在四個 allowed hosts 以外開始；即使 input、CPU affinity 與 model 都正確，
整個 attempt 仍必須保存並排除。較安全的最小做法是 wrapper 只列出明確的
`label=ssh-target` allowlist，claim 後再由 run metadata 驗證實際 hostname。

多台主機共用 NFS artifact root 時，不要讓平行 runner 首次同時執行
`mkdir -p common-parent/distinct-child`。Issue #166 consumer formal attempt 001 中三台同時建立缺少的
共同 parent，兩台在 CPAchecker 啟動前收到 `mkdir: Already exists`，只有一台成功。Recovery 是在
preregister 後、啟動 parallel jobs 前由單一 controller 建立空的 common parent；每個 runner 再用
plain `mkdir` 建立自己的 distinct child 並拒絕 pre-existing child。保留並排除整個失敗 attempt，
以新的 runner/input hashes preregister 下一個 attempt；不要把部分成功 case 混入結果。

# 準備文件不能自稱已取得執行核准

Worker 產生的 admission 範例必須保留 `NOT_ADMITTED`，不能預填 `issued_by_root=true`
或已核准狀態。Issue #208 曾在未經 root 驗收的 packet 出現這種狀態；root 保留原檔、
撤回假核准，修正 timeout/cleanup與 accounting後才寫真正綁定檔案hash的核准。
Prospective approval可以先涵蓋完整bounded工作，實際執行再確認該host既有批次terminal、
完整capture與owned程序清理及可用核心；不必等待其他不相干host或反覆要求人工核准。

# 探索性批次的單題結果與全域停止（Issue #208）

單題 native crash 若已有完整 raw exit/log、失敗分類且 owned process 已清理，可保留為
crash outcome，讓其他獨立題目繼續。Timeout、analysis failure 與 official wrong 也必須
原樣計數；不能因 smoke_ok=false 就推論 integrity_ok=false 或整批不能執行。
是否停批必須依事前記錄的實驗政策，不能為了完成率抹除失敗或新增結果導向 exclusion。

Root 曾把 Stock24 的單題 crash 修復誤設為全部剩餘題目的共同前置條件，造成完整24筆
已有 integrity PASS，剩餘工作仍閒置。修正是在 issue 明確記錄 prospective continuation，
保留舊 STOP receipt 和24筆原始結果，只執行未啟動的194筆 Stock；cohort、runtime、limits
不變。#215 的診斷與其餘題目執行解耦，沒有宣稱 crash 已修好或舊 stage 已通過。

派工需一次授予完整 bounded shard 的執行權與明確終止條件，不應每一 wave 再等 root。
單題結果只停止不可信的該筆 evidence；身份/hash不符、程序無法控制、持續資源失控或
provider/cost異常才停止受影響 lane，證實是共用問題再全域停止。已凍結 stop policy 若需
調整，先保留歷史、公開記錄調整與適用範圍，再啟動後續 cells；不可默默改寫原先 gate。

# Exploratory integration qualification（Issue #180）

Owner-authorized exploratory run 可接受普通背景 load；這只放寬 performance admission，
不豁免官方 labels、data model、raw exit/verdict 或 provenance。六個 crash/unqualified exclusions
與十二個 official-label wrongs 是不同集合，annotation 不得把 wrong 改成 correct。

在共用 runner 中，以單一 launch builder 將 frozen `ILP32`/`LP64` 映射為實際
`analysis.machineModel=Linux32`/`Linux64`，並把 exact argv 存入 execution sidecar。
只測 command string 不足；tiny width fixture 應以實際 verifier 證明兩種 model 的不同 verdict。
`run_one` 由 xargs 的新 Bash 呼叫時，必須 export 它呼叫的 builder/mapping functions。

Python capture 若擁有 outer wall timeout，argv 不應再套 GNU timeout；這樣 raw exit/signal
才來自 verifier process group。Log 應由 capture 以 exclusive creation 建立，不能先寫 runner
的 model marker，也不能在 crash 後補寫 UNKNOWN。SIGTERM timeout 後的 SIGKILL 仍有
process-exit race；捕捉 `ProcessLookupError` 後繼續 wait/save status，保留真實 evidence。

Runtime identity 會指紋化既有 classes、libraries、launcher、JDK executable/modules/libjvm；
大型 file-hash inventory 應經 temporary JSON file 傳遞，不應放進單一 environment variable。
Runner 行為測試若 mock verifier，也要在 temporary repo 建立明確的 fake runtime 與 Git HEAD；
不能偷偷依賴開發者 checkout 已 build。這種 fixture 只算 harness test，不能當 live evidence。

凍結 verifier worktree 可 detach 到 exact SHA，後續 PR 修正放在另一個 worktree。如此新的
review commit 不會改動正在執行的 HEAD、classes 或 harness bytes。若只修正控制器而不中斷
既有 verifier draws，保留舊控制器與 freeze，建立新的 version/freeze，明列接手邊界；不得
silent overwrite。停止條件必須使用 recorder 的實際 taxonomy，例如 `out_of_memory`，
不要用沒有產生過的 `oom` alias。

參考：integration `7b69e3c`、review fixes `7dc36a3995`；Issue #180/#182。

### Refinement attempts 不等於 completed dump rows

`PredicateCPARefiner` 在檢查 counterexample 前增加 refinement count；
`VGuideRefinementBridge.onSpuriousAfterRefinement` 只在 spurious strategy 成功返回後
寫 `refinements.jsonl`。因此 summary 的 N attempts 與 N−1 rows 可合法來自最後一個
feasible counterexample、timeout 或 analysis failure。不能以嚴格相等斷言判定檔案損壞；
缺口應保守標 partial coverage，再以原始 log 解釋，partial dump 的候選計數僅是已觀察列。
R>N 則不可能，應拒絕；不能把缺失觀察補零。

已驗證：PR #184 `5f84e2387f` 的 regression tests，以及 #108 獨立 producer 語義檢查
（Issue #180 comment 5554823395）。同一修正也拒絕會在 launcher 尾端追加的
`CPACHECKER_ARGUMENTS`，並要求 `score_wall_s` 從 uncapped raw statistic 重新計算。
Xargs 產生的普通 Bash 不繼承 parent `set -e`；function/command substitution 的模型驗證
錯誤必須逐層 `|| return 1`，不能只靠 export helper 或 parent manifest precheck。

### First-spurious round cap 不等於 transport call cap（2026-09-06）

Frozen `37064e4` 的 `VGuideRefinementBridge` 在 SAFE primary 沒有可解析候選、
但有 rejected text 時，仍可在同一 round 再發一次 repair call；`LlmCallScheduler`
只數 completed rounds。故 FIRST_SPURIOUS / samples=1 最多是一個 scheduled round，
不是保證一個 logical provider request。每 request 允許兩次 retry 時，structural
worst-case 是每 task 2 logical / 6 HTTP attempts。正式成本 freeze 必須區分預算、
程式可達上限與觀測 calls；不能從 round count 推論 dollar/call cap。
#180 recovery 的 132 sealed rows 全是 `safe_primary`、repair=0；觀測 tokens 不受影響，
但舊 freeze 的218/654宣告不是 enforced worst-case。補跑 budget 要重新明確確認。
證據：Issue #180 comment5559713563 / #182 comment5559713668。

### 跨主機 accounting 的統計範圍（2026-09-08）

報告寫在某個 host 的目錄、檔名叫 `accounting-final.json`，都不代表內容只涵蓋該 host
或整個實驗已完成。#208 的共用 reporter 固定掃描三台，top-level `counts` 是全域快照；
單機結案必須先用 `row.host` 篩選再加總，並分開記錄 `launched`、`terminal` 與預期題數。
曾把全域 157 次 HTTP 誤列為 Valkyrie 單機用量，實際該機只有 67 次；原始快照保留，
另存 host-local receipt 更正範圍。全域計數只能對全域上限，單機計數才對單機上限。
Task coverage 只計唯一 task records 或 `logs/*.execution.json`；外層 launcher 的
execution sidecar 不是額外一題。候選的 validated/injected 統計也不能改稱原始候選數
或證明 usefulness；未知 usage 維持 unknown，不得補零。

### Terminal sentinel 不是通知 callback（2026-09-06）

#177 monitor 在 STOP 出現時 exit0，早於九分鐘後的 final summary，且實際 command
沒有 `ping-peer`。Completed HAPI job 不保證 coordinator 被喚醒。Terminal wrapper
必須等待 real supervisor 完成 summary，再傳送 summary hash/counts/exit/STOP reason，
保存 notification receipt；accepted 只代表 CLI 接受，不等於 agent/human 已讀。

### Dataset lineage 與交付狀態（收尾查核）

#177 核對的現行 cohort 是224父集合按原序排除6題後的218；historical764
是舊研究集合，不能混作本次分母。六題排除與官方標籤判錯是不同分類。
重新納入題目需另立有資格驗證的 cohort，不能因 native crash 修掉就改寫既有218。

Agent 已交付、PR review 通過、PR merged、issue completed 是不同狀態。
#178/#179 的修正經整合 PR 吸收時，逐項比較剩餘差異；不要整批再次套用舊
runner/resume 語義。研究實驗得到可驗收的負結果可結案，但不代表相關程式
已合併或應 rollout。已凍結 runtime 與最新 PR HEAD 仍須分開記錄。

#109 context 修正與 prompt wording A/B 是獨立 intervention；沒有包含前者
的全量 run，不能用來宣稱前者有效。測試來源 context 時保持其他條件固定。

### Candidate 診斷的分母與資料層（#45）

Recovery 的218個 cohort rows 僅有184 matched pairs；132個 complete-response
rows 的 endpoint intersection 是21 ok、98 timeout、13 analysis_failure。
不能把全部 augmented 的28 ok 除以132。候選 entries 與注入 target placements
也不是相同單位，後者可以多於前者，不能直接當作唯一候選通過率。

`details.json` 沒有 per-location 欄位，不代表原始資料沒有：已直接核對的
sample 在 `llm_rounds.jsonl` 的 `response_raw` 內有 `candidates[].loop_heads`，
`refinements.jsonl` 有 `validated_predicates[].loop_head/injected`。區分摘要
丟失欄位、原始 schema 缺欄位與讀檔 I/O 阻塞；sample 成功不等於全 cohort
可讀或已彙總。Provider/model 資訊也應交叉查 launch freeze／config 等來源，
不可僅因 task-level details 缺值就宣稱整個 frozen experiment 沒有 metadata。

### 相依 PR 的 base branch 也是整合條件（2026-09-07）

#203 整合時，#195 的 head 含 #191，但實際 `baseRefName` 是 #191 的來源
branch。合併 #191 後使用 `--delete-branch`，GitHub 產生 `base_ref_deleted`
並自動關閉 #195；這不是測試失敗或人工否決。單看 head ancestry／MERGEABLE
不足以驗收相依 PR。合併前列出所有 open PR 的 `baseRefName`，先把後繼
PR 改接 main，再刪除父分支。#196 對 #184 有相同依賴，已用此順序避免。

GitHub 不允許直接改 closed PR 的 base。復原方法是將已合併父分支恢復到
原 exact head（無 force），reopen 後 retarget main，再驗證 head、diff、checks。
重新開啟可重啟 Swear Review；舊 pass 不能當作目前 IN_PROGRESS 已完成。
測試報告也要區分父 PR 的獨立 test tree 與後繼 PR 的合併 test tree：
#191 原有 7 個專用測試，#195 合併後有 10 個，不能把後者寫成兩個 head
各自都跑過 10 個。

### PR99 歷史 manifest hash 不可推定等價（2026-09-07）

#207 離線盤點確認 canonical 218（`3350720a…`）與224父集合排除六題後
逐筆欄位及順序一致；PR99 宣告的 `a969cad2…` 原始 bytes 未找到，差異
仍為 unknown，不能自行歸因於 JSON 排版或 metadata。舊 generator、六題
list、label annotation 與 provider helper 沒有 current-main caller，因此
採保留 branch／artifacts、退休舊 PR 的處置，不為了清 PR 搬入無人使用的
工具或以 annotations 豁免官方 wrong verdict。

### first_spurious 下先證明 ablation 變因會生效（2026-09-07）

在合併後 main `1eef72b44e`，`LlmCallScheduler.shouldCall` 的 FIRST_SPURIOUS
只接受 refinement 1。`VGuideRefinementBridge` 在 scheduled call 前建立 CE
history／completed refinement outcomes；history 是 build 後才 record 当前 CE，
outcome 是 after-refinement 才完成，故首輪通常都是空。history store 的 record
本身也位於 scheduled-call 分支，不能假設涵蓋所有未呼叫模型的 native rounds。
LLM ownership 初始空且 lastValidation 每次 before-refinement 清空；單次注入
通常無上一輪 owned predicates 可替換。#40/#42/#43 要先用離線 evidence 證明
多輪下 nonempty history/outcome／owned removal，再花錢做 ablation；#41 則核對
實際 native exposure。這是實驗變因是否生效的條件，不是現在改預設 schedule
的理由。多輪前仍須處理 successful-round counter 不等於 hard attempt cap。

候選 entries 比 target placements 少可以只是 multi-head 展開，不能單憑
1,418／1,469 差異宣稱 validity bug。優先從 raw JSONL 產生 report-local
(task, round, response-index, candidate-index) occurrence identity，不為了摘要
缺欄位立即新增 production schema。新 main 的整體對照與舊 runtime pending
cells 是不同 estimands；#208 的新 comparison 不可填進 #180 缺失 cells。

# SSE 完成與候選 JSON 成功是不同層（2026-09-08）

在 #208 / #221 的既存證據中，8 個回應即使記錄 HTTP 200、`stream_success`
與 `[DONE]`，保存的 content 仍是未完成 JSON；另 1 個回應在 client 以
`No text content in LLM response` 結束。`PredicateProposalClient` 先完成 SSE
解析才寫 cache，bridge 又在 client 成功返回後才記 LLM round，因此
`llm_calls=0` 或沒有 cache 並不證明沒有 HTTP request。先對照 per-attempt event，
再分別判斷 transport、candidate parser、predicate validation 與 solver outcome。

凍結版本未保存 `finish_reason`，不能從這些 bytes 推定 token cap、拒絕或截斷原因。
usage 缺失仍是 unknown，summary 的 observed-only 零值不是零消耗。
後續觀測修正應保存有限的 terminal metadata 與 parser reason；不需要保存完整
HTTP headers 或憑空補齊 usage，也不得為修正報表重跑或替換原始樣本。

## 非空 verdict 不等於已解決（2026-09-08）

`UNKNOWN` 是非空字串，不能用 `bool(verdict)` 或「非空數減 correct」算 official wrong。
#208 Cthulhu 的舊 callback 將 stage24/remaining 的 wrong 寫成 4/33；此算法恰好
重現該誤報，但原始 records 經既有 `check_core_only_smoke.summarize` 分類實為 1/1。
整合必須共用 validator 的 official verdict 分類，並核對互斥的
correct + wrong + unresolved = manifest 題數。這是報告計數更正，不是新的實驗結果。

## Counterfactual binding identity 與既有 dump 語意（2026-09-08）

#224 Problem14 的 raw response ordinal 不等於 production dump `predicate_id`。
離散候選是 response ordinals 1,2,3,10,11,12，卻是 dump IDs 1..6；數值切點候選是
response ordinals 4..9，卻是 dump IDs 7..12。做 group-removal 必須凍結
request/response、head、formula 與 ID mapping，不能把不同 namespace 的整數混用。

不要從區域變數名 `injected` 或「同一 list 傳兩次」推論 telemetry 丟失 validation。
已逐函式核對：`markInjected()` 保留全部 validated records，只設定各項的 injected flag；
`validatedPredicatesJson` 輸出全部，`injectedPredicatesJson` 才過濾 false flags。
#226/PR227 的 replay-only suppression/filter 已重用這條既有路徑。先追產生、轉換與消費端，
再判定資料是否遺失；不為錯誤的初步診斷新增重複欄位或框架。

`vguide.replayInjectionMode` 支援 FULL（預設）、SUPPRESS_ALL、EXCLUDE；非 FULL 必須
使用 `VGUIDE_LLM_REPLAY_DIR`。EXCLUDE 的 `vguide.replayInjectionExclusions` 每行是一個
`head=N<number>;formula=<canonical SMT from dump>;provenance=<source_profile>`，不是 raw
candidate 文字或 response ordinal。匹配要跨 refinement rounds 累積，不能逐輪把尚未出現
的 selector 當成不存在；正常 analysis end 才檢查未匹配集合。未進入 dynamic selection 的
run 不會觸發此檢查，不能視為合格的 exclusion。真正的消融須逐 run 驗證 request/response、
完整 validated bindings 和實際 injected flags；mock callback 或設定值不等於實際介入成功。

## 實際 benchmark input 與同名 source 不可互換（2026-09-08）

#225 驗收發現，`mapsum5.yml` 與 `sep.yml` 的 `input_files` 分別指向
`mapsum5.i` 與 `sep.i`；frozen record 的 `source` 和 `source_sha256` 也對應這些
預處理輸入。同目錄雖有同名 `.c`，其 bytes/hash 不同，不能把 `.yml` 副檔名
直接改成 `.c` 再宣稱是已執行的 source。

建立證據索引應沿 task `input_files` 與實際 record `source` 解析路徑，再驗 bytes/hash。
同名原始 `.c` 若有助閱讀，可另外標成 supplemental source；不可替代執行輸入或
混用兩者的行號、hash 與 prompt visibility。這是報告 provenance 更正，不是新實驗。
