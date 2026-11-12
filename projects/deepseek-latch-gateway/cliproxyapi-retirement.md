---
title: CLIProxyAPI 接替自製 DeepSeek Gateway，Swear Review 繼續使用
scope: projects/deepseek-latch-gateway
status: active
updated: 2026-11-12
---

- 只有自製 deepseek-latch-gateway 封存。使用者明確糾正：Swear Review 必須繼續使用新 Gateway；先前將 Swear Review 封存／停服務是 agent 的誤解，不是使用者要求。Swear Review repo 已解除封存，Oracle swear-review／cloudflared-swear 服務已 active/enabled，本機與公開 healthz／readyz200（公開探測使用正常 User-Agent；預設 urllib UA 可能被 edge403）。
- 改用官方 CLIProxyAPI v8.0.53，不維護自製 proxy／plugin。transfer_MAC PR80 保存模板、私人設定 renderer 與 Windows launcher；shared-skills PR56 曾誤移除 Swear，PR59 merge40498da 恢復 Swear 優先、Gemini備援與最新 head review gate。
- 八個本機 Gateway 實例已部署並驗證：Mac、mazu、cthulhu、valkyrie、zeus swear01、su_zeus swear02、Oracle、swop。Athena 送修排除；共享 NFS 檔案變更不等於驗證 Athena。一般 localhost port35001，su_zeus35002。Oracle 是 aarch64，不能裝 amd64 binary。
- 遷移當時，公開模型只有 deepseek-flash／deepseek-pro；四個同等 slot 各一把 key（OpenCode1/2、CommandCode1/2），新對話輪替，同一對話固定，失敗後重新綁定成功替代 key並冷卻。上游沒有跨供應商通用 latest alias，推出新型號仍需更新兩個官方設定 mapping，不能承諾永遠自動選到最新版本。

## 2026-11-12：Command Code 第三槽

- [transfer_MAC PR #88](https://github.com/swear01/transfer_MAC/pull/88) 加入可選的 `command-code-3`，供 Flash 和 Pro 使用。這是 CLIProxyAPI 的部署設定，不是修改或重建 router 本身。Unix 由私人 keybag 的 `COMMAND_CODE_API_KEY_3` 產生設定；Windows 由既有 SYSTEM 排程讀取機器範圍的 DPAPI blob。沒配置第三槽時保留原路由與重試上限。
- 8 個 CLIProxyAPI 實例都核對了第三槽與實際推論。第三把 key 直接打 Command Code 的 Flash／Pro 都成功；SWOP 核對 DPAPI 載入後的設定、模型清單和 Flash 推論，沒有另測 SWOP 的 Pro。Mazu 恢復連線後，用既有程序的日誌確認第三槽已出現，Flash／Pro 也成功，因此不必為了驗證而重啟忙碌的 router。
- 若 SWOP 的公網 SSH 不通，先查 Tailscale 是否在線。使用 Tailscale 位址連線時，可用 `HostKeyAlias` 指向已信任的 SWOP 位址，保留嚴格主機金鑰驗證；先確認指紋吻合，再透過 SSH stdin 傳遞憑證，避免把 key 放進命令列或 agent 訊息。跨主機僅記錄 Vault 項目名稱與部署結果，實際 key 留在 Vault 和各機的私人設定。

## 需求邊界

- 使用者只要淘汰自製 Gateway；Swear Review 是保留的呼叫端，不是要淘汰的元件。後續換供應商、改 key、更新 Gateway 時仍須保留 Review 服務、專用 tunnel、GitHub App 與 review 規則，除非使用者明確另改需求。
- 停用與封存前的範圍核對流程見 [全域替換範圍規則](../../global/workflows/config-change-scope-discipline.md#停用封存前核對範圍)。

## 憑證與呼叫端

- 舊 OpenCode slot2 是使用者確認的假值；新 slot2 使用原 slot3。slot 是配置名稱，不能視為永久帳號 identity。
- Vault 的一般項目 API · deepseek 現為 local-gateway marker；官方 DeepSeek／OpenRouter 改為 _service/providers 下的明確 official 名稱，四個目前上游項目放 _service/deepseek-gateway。舊 Command Code 項目與現用 key 不同，保留於 _archive/providers，沒有覆寫或刪掉舊 credential。
- Unix 真 key 統一保留在 <home>/.secrets（0600）：OPENCODE_API_KEY_1/2、COMMAND_CODE_API_KEY_1/2、DEEPSEEK_OFFICIAL_API_KEY、OPENROUTER_OFFICIAL_API_KEY、OPENAI_API_KEY 使用 bare assignments；Gateway renderer 靜態讀取此檔，不 source/eval。原 <gateway-config-dir>/secrets.env 已移到私人 archive。DEEPSEEK_API_KEY／DEEPSEEK_GATEWAY_API_KEY=local-gateway 與本機 BASE_URL 仍 export；其他工具 export 保留。
- Keybag 開頭暫停 allexport，unset 舊 export 屬性後才賦真 key，結尾還原呼叫端 set -a；只刪 export 字樣不足以應付 pi-safe／HAPI 的 set -a loader。bash／zsh 三種繼承情境已驗證，公開範例另測 POSIX sh。HAPI Hub setup 只在服務專用子 shell 讀 keybag，再原子產生0600 hub.env；移除 NFS .bashrc 的 OpenAI 全域 export，現有 Hub 語音 env hash 保持。改名、未 export 與0600不隔離同帳號 agent。
- 同時保留上游 x-opencode-session 與 Gateway 可識別的 X-Session-ID／X-Session-Affinity。只有 x-opencode-session 時會落到內容 hash，不能證明對話固定。Pi 兩個公開模型的 compat.sendSessionAffinityHeaders=true、sessionAffinityFormat=openrouter 可產生 X-Session-ID；不要改其他模型／預設值。
- 隔離測試必須讓不同 session 送完全相同內容，並讓同一 session 的歷史增長、compaction 後仍固定；僅重送同一內容會把 hash fallback 誤判為 session affinity。

## 部署經驗

- Mac 上直接覆寫執行中 binary 的同 inode 後，新程序曾 SIGKILL，即使 checksum／codesign 驗證通過。重新從已核對 archive 寫入新 inode，再原子替換後恢復；同一官方 hash，沒有重簽。AMFI cache 是推測，已證實的是換 inode 的修復方式。
- Windows 使用既有 SYSTEM task、機器 DPAPI 與 SYSTEM／Administrators ACL。先限制父目錄含繼承 ACL，再建立或開啟私人 config，避免其他使用者先持有 read handle；管理員 provisioning 與 SYSTEM consuming 的情境不能只把讀取端改 CurrentUser。
- Windows launcher 直接使用 canonical config.template.json 原始變數，不能另維護一套 placeholder／filename 卻未保存 provisioning 轉換。
- 清理檔案不會移除已運行 HAPI Runner 的 process.env。Mac／Oracle wrapper source 清理過的 .secrets；Linux須改每台本機 runner.env，再只重啟 Runner 主程序。KillMode=process、AbandonProcessGroup=true、PM2 treekill=false 保留子 session；本次6個仍存活的 session roots 全部保留。既有 agent 本身仍持有舊環境，只有新 agent 保證乾淨。
- Zeus swear01 的 legacy runner 與 swear02 指向同一 runtime envfile但 owner1075，前者既有 activating；沒有為這次 Gateway 移轉修理／重啟它，勿把這項 baseline 問題當新部署成功或全 fleet runner 健康。

八實例均實際驗證 Flash／Pro HTTP200且非空、官方 runtime版本/hash、loopback、未認證401、管理API404、Pi session設定。一次暖機樣本回報 cached2176/input2272；不是所有對話的命中率保證。Mac一次短 token-cap Pro回200但空，單次重試正常，保留證據；不要把HTTP200單獨當完成內容證明。

## Source size measurement

- Official v8.0.53 / main commit 01b24f1f95dc9b309765212c52b25b25e005ff01: cloc 2.06 measured production Go 274,919 code lines (827 files), _test.go 418,862 (1,174 files), all Go 693,836 code / 789,490 physical lines. Runtime single-binary simplicity does not imply small source. 2026-11-04 GitHub API: 57,415 stars / 8,794 forks; active-user count unavailable.

## Swear Review restoration

- Runtime review.auto=false、gate.mode=off、OCR1.13.5/concurrency4、llm.model=deepseek-flash、URL localhost35001/v1/chat/completions，專用 env 的 OPENCODE_GO_KEY 改用 local-gateway。不要把健康端點200或gateoff check success單獨當LLM或無findings證據；需讀實際review結果／inline findings。PR59 job1695確實經webhook與publication成功，但OCR無selecteditems而skipped，不能稱其已實際打LLM。
- Verto local trigger-pr-review helper從pre-retirement備份逐byte還原為Swear，sh-n通過；舊誤退役期間取消的PR56 review對應PR現已merged，沒有擅自重跑其他進行中的PR。

- 復原完成：shared-skills PR59 final40498da 恢復規則且8個aliases逐target驗證，transfer_MAC PR82 merge4f6647e 用 git mv 恢復 active runbook 與指標；Swear Review PR19 mergeb7bd7b8 加入同reviewUUID的 X-Session-ID/nativeheader並拒絕configuredaffinityoverride。PR19 exacthead61f6f05 經正式Swear job1696/run1653實際DeepSeekreview completed，1項可選maintainabilityfinding經判斷回覆resolved，非skipped；已觀測OCR→localhost35001連線與marker/model設定。
- 使用官方 scripts/deploy-oracle.sh b7bd7b8 部署，151tests/typecheck/build於正式部署流程通過；服務新PID與啟動狀態、已部署dist兩種header、公開／本機healthready、DBintegrityok、舊gateway仍archived與Swearrepo未archived均核對。不要為換Gateway而退役使用它的應用程式；本次是誤解範圍後的修復，歷史取消review不自動補跑。

- 2026-11-11 加入 NVIDIA NIM 備援（priority -1）與 400 insufficient-credits failover 規則：見 [NIM 備援](nvidia-nim-fallback.md)。模板已由 transfer_MAC PR #85–#88 同步。
