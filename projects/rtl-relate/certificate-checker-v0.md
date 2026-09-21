---
title: AIsimpV RTL certificate checker 的已驗證界線
scope: project
project: AIsimpV
status: active
updated: 2026-09-21
---

# AIsimpV RTL certificate checker

獨立專案位於 `<project-root>/rtl-relate`，不 import NeuroAbs。目前人工 P1 counter 與 P5 stalled producer 可跑真實 Yosys → BTOR2 → typed IR → Z3 certificate gate、free-nondeterminism property checking、exact concrete replay。既有 tiny 回歸保留；2026-09-21 已另完成四題公開 RTL／FSM 人工改寫矩陣，但未證成驗證加速。

## 需要保留的 soundness 防線

- Frontend 不可丟掉 clock identity。實際反例：C 用 clk，A 改 other_clk，在模型未保存 clock 時仍通過 frozen clk contract。修正是 model 保留經 frontend 驗證的 name/edge，checker 與 contract 精確比對；真 RTL 回歸確認正常 case ACCEPTED、錯 clock ERROR。
- 五項義務必須共用同一 w。P1 observer 不讀 z，無法單獨測到此漏洞；另用 Mealy fixture 令 STEP_MAP 要 z=0、OBS_MAP 要 z=1，確定不存在共同 witness。
- J=(x<2) 搭錯 w=0 可讓 P1 STEP_MAP 成立，但 STEP_J 必須拒絕。空 initial set 也會令所有 implication 空泛成立，必須先獨立檢查非空。
- Property 不收 h/J/w；每拍 z 使用 fresh symbols。tiny backend 的 SAFE 來自完整一步證明或完整 reachable closure；有限探索達上限回 UNKNOWN。公開 RTL backend 的 SAFE 需要 SMTBMC base 與 k-induction 都通過，單獨 BMC 通過只能是 BOUNDED。
- Exact replay SAT 只表示 prefix FEASIBLE，還須確認違反 frozen property 才是 BUG。P5 bug 的第一條 abstract CEX 可能 spurious；若另由 concrete search 找到不同 trace，必須明示其來源，不能冒稱原 trace 已具體化。
- Concrete fallback 的 SAFE/UNKNOWN/ERROR 也要算成本；不能只在找到 bug 時計費。Gate rejection 與 exception 也保存耗時。

## 可重現工具路徑

Checker 僅 Python standard library + Z3 CLI。YoWASP Yosys 可隔離安裝在 `<task-worktree>/.tools/yosys-venv`，不需要 sudo。已測 yowasp-yosys 0.69.0.0.post1233、Yosys 0.69 / 9f75ca1f9、Z3 4.15.4。

重跑介面：`python3 -m unittest discover -s tests -v`；`python3 -m rtl_relate demo --out results/<new-run>`。每次用新目錄，保存 hashes、SMT queries、raw logs、RTL exports、traces 與成本。

2026-09-21 正式人工矩陣 `wednesday-20260921-final01`：FSM、skid8、skid32、pipeline32 共 12 基本 attempts，全數符合 good SAFE／coarse SPURIOUS_TRACE／錯證書拒絕。FSM one-hot→binary 以非平凡 J 限制合法狀態，正反向證書均通過；skid J 為 r_valid ⇒ output_valid。公開部分只有 2 個家族、3 個實例，不能當成 4 個公開家族。重跑介面：`python3 -m rtl_relate.wednesday --out results/<new-run>`。LLM 固定 C/A 與 joint 兩條 pilot 各最多 4 次／900 秒；其結果需查獨立 ledger，不可把人工證書成功歸給 LLM。

## 外部 benchmark 接入陷阱

- `aman-goel/avr` 固定 `9a76dc632066c4416cebccda3a4974a4f8adede8`：`tests/opensource/h_Arbiter/main.v` 的 client 把 `rand_choice=0`，且 `req=0`、`state=NO_REQ` 初始化，請求永不發生；不可當作非平凡正例。改為自由輸入會改變 benchmark，須另標變體。
- 同版 AVR pipeline 的實際檔案是 `tests/opensource/pipeline/pipeline.v`：96-bit datapath 加 64-bit monitor。原生 assertion 的一般 export 會遇到 `$check`。目前只對固定 source hash 接入受信任 extraction：保留原 history monitor，匯出原 prop 檢查等價，再以同一 frozen predicate 建立外部 harness。不可一般化成刪掉任意 assertion。
- `ZipCPU/wb2axip` 固定 `2e8d3bc2d26ddc33d1881022a2a2b9d3f0c16b9b`：原始 `rtl/skidbuffer.v`、預設 `DW=8, OPT_OUTREG=1, OPT_INITIAL=1`、未定義 `FORMAL` 時，現有 frontend 匯出 18 state bits，`i_reset` 為普通 public input。v1 明確把同步 reset 保留為 unconstrained public input；實驗驗證衍生 hold property，不載入上游完整 `FORMAL` assumptions，不能宣稱完整 upstream suite 已通過。原始 `rtl/sfifo.v` 則為 `UNSUPPORTED: $memwr_v2`；預設非同步讀取配置的 memory 未初始化，不可補零來讓接入通過。

## 公開專案與協作入口

公開 repo 已依工作資料夾名改為 `swear01/AIsimpV`；舊名會 redirect，remote／active links 已更新，改名後也成功 `gemini-link swear01/AIsimpV`。本機子目錄與 Python package `rtl_relate` 沒有為此做無關改名。`docs/wednesday_plan.md` 是新排程，issues #1–#12 保存依賴／檔案責任／驗收證據；#1 總追蹤、#2 固定來源與 contracts、#12 為不計入原四題分母的 FIFO。

GitHub Actions `Verify` 已在 Ubuntu 24.04 / Python 3.12 實跑 89 項測試與完整 8 列 demo，保存 `verification-evidence` artifact；不是只有本機 3.14 結果。用 `unittest ... | tee ...` 保存輸出時，job 明設 `defaults.run.shell: bash` 以啟用 pipefail，避免 tee 遮住測試失敗。

## 公開實驗的量測與工具界線

- `h/J/w` 不進 property harness。skid 18→10／66→34 design bits；pipeline 96→64 design bits，64 原 monitor bits 不變。COI 後的 bit/cell 數還含取樣 harness，不能冒稱是純 DUT 規模。
- 正式 run 的 good workflow 約 1.43–1.92 秒，正常 B0 約 0.54–0.60 秒，單次結果沒有加速；人工準備時間未量測。失敗 attempts、reverse certificate、cover/strictness/replay 都算 machine cost，但工程開發回歸不混入該正式矩陣。
- Yosys Witness `bits` 字串須反向對應 LSB-first signal IDs。由 actual input witness 與 typed model 還原輸出後，仍須重新檢查 A-prefix SAT 與 frozen P 違反，再 exact replay C；缺漏 input bit 的 zero completion 必須顯式記錄且重新驗證。
- SMTBMC `requested_depth` 不代表已完成檢查深度；CEX 保留實際 failure step。frontend／formal 子程序用總 deadline，timeout 要終止整個 process group，不能每個階段重領完整總預算。

## Codex pilot 的實際結果與隔離教訓

- `codex sandbox` 的 OS probe 通過，不代表真正 `codex exec` 的 Code Mode 工具也能啟動。CLI 0.154.0 在本次環境的巢狀 bwrap loopback 出現 `Failed RTM_NEWADDR: Operation not permitted`；停用 Code Mode 又令依賴它的工具不可用。固定 C/A 的四次全部耗在 infrastructure，零有效證書探索，不能歸因模型不會找 mapping，也不能重置 attempt ledger 隱藏失敗。
- 可用較小權限的替代傳輸：將同一份經審核 bundle 與 canonical hashes inline 放入 fresh prompt，停用工具，只接收嚴格 JSON schema 的 RTL／certificate 字串；parent 原樣 materialize，再由 frozen verifier 獨立檢查。這是 transport amendment，不是人類提供 h/J/w。原 ledger bytes 保留，修訂與人工介入可另用 sidecar 綁其 hash。
- 此 joint pilot 兩次內成功，約 86.784 秒。第一份 RTL 已有 10 state bits，但證書 abstract hash 錯；模型依真實 export feedback 自行修正證書，同 RTL 經 gate ACCEPTED 與自由 z 的 SAFE proof。A 保留 r_valid/o_valid、刪 r_data，h 是保留 state 的 projection，J=true、w=r_data；並非人工 occupancy 重編碼。原始兩候選獨立重驗重現同分類，不能因此主張 feedback 因果效果、普遍自動改寫或加速。
- 凍結 evaluator v3 的 Python API 需要 absolute Paths；relative output 會因 worker cwd 被重新解讀。新版 evaluate 在共同入口 resolve snapshot/candidate/out；不要覆寫已用於實驗的舊 snapshot。親自 recheck 時應驗證具體 phase／reason，不能僅看到預期 ERROR 就算重現。

## 執行邊界的後續驗證

2026-09-21，程式 head `788dce811f420dd64cd53e1fe672e94afe6600f5` 的 163 項本機測試與 GitHub CI 通過；工程回歸不算研究 attempts。

- 殺掉 process group 後，無期限 `communicate()` 仍可能卡住：新 session 的 descendant 可持有 stdout/stderr pipe。真實回歸重現約 15 秒等待；修正是立即記 timeout、kill 後最多再收集 1 秒、保留 TimeoutExpired 的累積 bytes、關 pipe 並 poll direct child。不要把真正的 PermissionError 一律吞掉。原始 stdout/stderr 要先以 bytes 保存，再嚴格解碼，避免格式錯誤消失於記錄之外。
- Network probe 只接受 PermissionError 的 EPERM/EACCES；timeout、connection refusal 或 offline 都不能證明 sandbox 拒絕連線。
- 一旦 generation 偵測到 trusted input 篡改，該 attempt 永久只能接受 infrastructure-error feedback；外部稍後還原檔案，也不能改報 SUCCESS。有 candidate hash 時仍必須核對原始輸出。
- Runtime 修正使用另一個經審核的 snapshot 路徑，保留舊 snapshot。`load()` 先核對全部舊 trusted hashes，再把新 snapshot 加入 ledger；不需也不應提供 skip-validation。Synthetic ledger 驗證新路徑可加入且原四次 attempts／費用不變，修改舊 snapshot 則先被拒絕，ledger bytes 保持不變。

## 固定 pair 跟進與有限 FIFO 評估

- 在保留原四次 infrastructure failures 後，另行於生成前預註冊 `skid8-certificate-inline-20260921`，使用 merged `9dd8e24` 與相同人工 occupancy C/A/contract、既有 gpt-5.6-sol/high。首個原始候選即 ACCEPTED，free-z property SAFE，charged 38.610 秒；乾淨 git archive 重驗六份 query hashes 與 property artifacts 全相符。h.n 為 r_valid ? 2 : (output_valid ? 1 : 0)，J 為 r_valid ⇒ output_valid，w.z 為 r_data。這是固定人工 A 的對應發現，不是自主產生 occupancy RTL；沒有 repair，不證明 feedback 因果收益。詳見專案 `docs/reports/data/llm-certificate-followup*.json`。
- CLI JSONL 的 `item.type=error` 不必然是工具執行失敗。本次兩項出現在 turn.started 前，內容分別為 skip_host_skill_discovery 實驗功能警告與刻意關閉 Code Mode；後續 exit 0、turn.completed、正確 structured output、無 actual tool items。應按事件與實際輸出分類，不能把所有 error item 當 tool call；原誤判保留，另用綁 hash 的 diagnosis 更正，沒有追加模型呼叫。
- `evaluate` 雖會 resolve absolute paths，仍要求輸出目錄的 parent 存在；離線包重驗範例應先 `out.parent.mkdir(parents=True, exist_ok=True)`，並選新的 out。缺 parent 是 gate 前的操作錯誤，不是 candidate 或證書失敗。
- sfifo 固定 4×8、async read 的有限 memory_map 實測 42 state bits，32 memory bits 無 init、10 control bits 有 init。原前端拒 `$memwr_v2`；展開後先拒 disabled-write X branches，BTOR `-x` 引出的匿名 input 又先觸發命名拒絕，尚未走到既有 missing-init 限制。`opt_clean -purge` 可刪 `r_empty` 名稱，留下同 bit 的 `o_empty`；初值審核須看所有 physical-bit aliases，不能只查偏好的 net name。不得補零、聲稱 normalization 等義或 FIFO ordering 已證；本輪只完成 UNSUPPORTED assessment，獨立 script 為 `scripts/assess_fifo.py`。

## 固定候選的重複驗證時間

2026-09-21 timing runner 在 source `de5898204580b40b3237f2f7f9d1077f7f7d6f91` 預註冊後固定執行；報告為 `docs/reports/verification_timing.md`，issue #17、PR #18。五個候選各 2 暖機、8 正式配對；所有 50 pairs 都 gate ACCEPTED 且 C/A SAFE，獨立資料審核 8,171 checks、零差異。

- 公平比較需兩邊都取各自已驗 certificate export 的 normalized RTL，再經同一正常 property preparation。舊人工矩陣部分任務從 raw 開始，不能與新八次結果混算。C/A 執行順序各半，候選順序輪換；每輪模型／contract／證書 hashes 與工具版本固定，失敗不能從 median 分母刪除。
- 分開記錄完整 property wall、base＋induction subprocess wall（包含 SMTBMC／Z3 啟動，非純 solver CPU）、已知候選驗證流程。後者取 pair wall 減 C property 實際呼叫時間，包含前端／證書／A proof 及協調；僅加各 stage 小計會漏計協調成本。per-record 報表 I/O 另保留在 suite wall。既有候選的發現／失敗成本仍須另列，不能當成免費生成。
- C→A property medians 秒：FSM .587→.572、人工 skid8 .578→.592、skid32 .592→.611、pipeline32 .618→1.070、保存 LLM skid8 .579→.590。FSM 差異僅約 .016 秒且分散範圍重疊；本輪不支持可靠加速。40 個正式完整驗證流程全部慢於同輪直接 C proof；不應只展示 state-bit 減少。
- Pipeline 前處理 state bits 257→225，但 base subprocess .114→.564 秒、induction .094→.097 秒；完整 property 約慢 73%。兩邊 base 深度與 induction 成功步一致，cells 同為 20，A 新增 subtraction、SMT bytes 稍增。這定位慢點於 base 求解，尚未隔離自由分解／算術表示的因果，不能斷言單一 subtraction 為全部根因。後续優先挑正常 B0 仍困難的 properties，並用實際 proof cost 評估候選。

## 有求解成本的公開 B0 篩選

2026-09-21，issue #19 與 `docs/reports/baseline_screen.md` 保存 17 個參數化 tasks、26 attempts，包含失敗／timeout／BOUNDED；獨立資料審核 1,200 checks 通過。這次僅 native B0 qualification，沒有抽象或證書，不能宣稱加速。

- 同一 pinned wb2axip `sfifo.v`，保留 SFIFO define、同步讀、BW=32、LGFLEN=6/8、無 empty/full bypass；沿用上游 prove depth 4，以 SMTBMC/Z3 base＋induction 完整證明。64×32 與 256×32 的三次 fresh-process proof medians 分別 5.772／7.313 秒，induction medians 5.439／6.925 秒。這是同家族兩個容量实例，不是兩個家族；其他 engine 尚未比較。
- 原生 28 assert cells 保留（同步配置有 2 個 A constant）、0 assumptions，5 covers 未執行。Memory 只初始化 mem[0]，其餘 2,016／8,160 bits 保持未知；不得補零。原生 proof 路徑的成功不解決 certificate frontend 的 symbolic/partial init、memory 與 formal monitor 接入限制。
- Current Yosys 對 edge-triggered `$check` 要先 `async2sync` 再 `chformal -lower`。第一次三筆 prep ERROR 保留；修正後使用 clocked assertion 的 SAFE/CEX/no-assert 真實工具回歸。獨立 runner `scripts/screen_baselines.py` 預設從 catalog 取深度，FIFO 4、AVR 20；不要為湊時間提高 FIFO unroll depth。
- AVR `s1269b` 的 README top file 是 `_mod.v`；`Huffman_dec` 的 plain lookup 無 255，原 assert 組合恆真，不能當成困難成功題。正常 SAFE 過快的設計與 BOUNDED／CEX 也必須記錄；不能只保留符合目標時間的數據。
- Native timing 包含 copy/prep/base/induction，尾端 artifact hashes 與 result 寫入在 suite wall。不能把它直接與舊 contract-harness abstract timing 拼成加速比。下一關是凍結原有 properties 並接通人工 FIFO rewrite＋certificate，完整計費。
