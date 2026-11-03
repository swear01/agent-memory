---
title: CLIProxyAPI 接替自製 DeepSeek Gateway 與 Swear Review 退役
scope: projects/deepseek-latch-gateway
status: active
updated: 2026-11-04
---

- 自製 deepseek-latch-gateway 與 swear-review 的 GitHub repo 已封存；Oracle 的 swear-review 與 cloudflared-swear 專用服務均 disabled/inactive，DB、取消狀態與備份保留。後續勿再觸發 Swear Review。
- 改用官方 CLIProxyAPI v8.0.53，不維護自製 proxy／plugin。transfer_MAC PR80 保存模板、私人設定 renderer 與 Windows launcher；shared-skills PR56 merge 8f219872 移除 Swear、保留 Gemini 優先與最新 head review gate。
- 八個本機 Gateway 實例已部署並驗證：Mac、mazu、cthulhu、valkyrie、zeus swear01、su_zeus swear02、Oracle、swop。Athena 送修排除；共享 NFS 檔案變更不等於驗證 Athena。一般 localhost port35001，su_zeus35002。Oracle 是 aarch64，不能裝 amd64 binary。
- 公開模型只有 deepseek-flash／deepseek-pro；四個同等 slot 各一把 key（OpenCode1/2、CommandCode1/2），新對話輪替，同一對話固定，失敗後重新綁定成功替代 key並冷卻。上游沒有跨供應商通用 latest alias，推出新型號仍需更新兩個官方設定 mapping，不能承諾永遠自動選到最新版本。

## 憑證與呼叫端

- 舊 OpenCode slot2 是使用者確認的假值；新 slot2 使用原 slot3。slot 是配置名稱，不能視為永久帳號 identity。
- 四把 key 只在 <gateway-config-dir>/secrets.env（0600）與 renderer 產生的私人 config，變數為 OPENCODE_API_KEY_1/2、COMMAND_CODE_API_KEY_1/2。一般 <home>/.secrets 只提供 local-gateway marker、DEEPSEEK_GATEWAY_API_KEY、DEEPSEEK_API_KEY 與本機 BASE_URL；顯式 unset 舊 OC／CC／OpenRouter（包含歷史 COMMAMD_CODE_API_KEY 拼字）防止繼承。
- 官方 DeepSeek／OpenRouter key另保留在私人服務檔，不全域 export；其他服務的 OpenAI／ElevenLabs／HAPI token 未變更。改名、隱藏路徑與0600只能減少誤用，無法隔離同一 OS 帳號的 agent。
- 同時保留上游 x-opencode-session 與 Gateway 可識別的 X-Session-ID／X-Session-Affinity。只有 x-opencode-session 時會落到內容 hash，不能證明對話固定。Pi 兩個公開模型的 compat.sendSessionAffinityHeaders=true、sessionAffinityFormat=openrouter 可產生 X-Session-ID；不要改其他模型／預設值。
- 隔離測試必須讓不同 session 送完全相同內容，並讓同一 session 的歷史增長、compaction 後仍固定；僅重送同一內容會把 hash fallback 誤判為 session affinity。

## 部署經驗

- Mac 上直接覆寫執行中 binary 的同 inode 後，新程序曾 SIGKILL，即使 checksum／codesign 驗證通過。重新從已核對 archive 寫入新 inode，再原子替換後恢復；同一官方 hash，沒有重簽。AMFI cache 是推測，已證實的是換 inode 的修復方式。
- Windows 使用既有 SYSTEM task、機器 DPAPI 與 SYSTEM／Administrators ACL。先限制父目錄含繼承 ACL，再建立或開啟私人 config，避免其他使用者先持有 read handle；管理員 provisioning 與 SYSTEM consuming 的情境不能只把讀取端改 CurrentUser。
- Windows launcher 直接使用 canonical config.template.json 原始變數，不能另維護一套 placeholder／filename 卻未保存 provisioning 轉換。
- 清理檔案不會移除已運行 HAPI Runner 的 process.env。Mac／Oracle wrapper source 清理過的 .secrets；Linux須改每台本機 runner.env，再只重啟 Runner 主程序。KillMode=process、AbandonProcessGroup=true、PM2 treekill=false 保留子 session；本次6個仍存活的 session roots 全部保留。既有 agent 本身仍持有舊環境，只有新 agent 保證乾淨。
- Zeus swear01 的 legacy runner 與 swear02 指向同一 runtime envfile但 owner1075，前者既有 activating；沒有為這次 Gateway 移轉修理／重啟它，勿把這項 baseline 問題當新部署成功或全 fleet runner 健康。

八實例均實際驗證 Flash／Pro HTTP200且非空、官方 runtime版本/hash、loopback、未認證401、管理API404、Pi session設定。一次暖機樣本回報 cached2176/input2272；不是所有對話的命中率保證。Mac一次短 token-cap Pro回200但空，單次重試正常，保留證據；不要把HTTP200單獨當完成內容證明。
