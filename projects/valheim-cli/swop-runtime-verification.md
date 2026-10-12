---
title: ValheimCli SWOP runtime and release verification
scope: projects/valheim-cli
status: verified-with-limits
updated: 2026-10-12
---

# SWOP 實機結果與限制

## 目前交付與實測：2026-10-12

- C# Bridge / Thunderstore **0.5.0**、原生 Rust CLI **0.6.0**，兩者版本獨立。GitHub main `db97d123151fe8dce41930251917eef8db2f0d26`；PR #13 已合併，四平台 CI run `38168482002` 通過。
- [Thunderstore ValheimCli](https://thunderstore.io/c/valheim/p/swear01/ValheimCli/) 已上架 0.5.0；Mods / Tools / Client-side / AI Generated 分類已核對。公開 ZIP 與實測包雜湊一致。主列表曾短暫快取舊版本，版本頁與更新後 API 已確認 0.5.0。
- [GitHub v0.6.0](https://github.com/swear01/valheim-cli/releases/tag/v0.6.0) 已由 prerelease 改為正式 **Release / Latest**，latest API 讀回確認；9 個附件下載雜湊核對相符。原生 CLI 提供 Windows x64、Linux x64、macOS ARM64 / x64；不是所有平台都已遊戲實測。
- 模組新增同一 loopback listener 上的 HTTP/1.1 `POST /v1/command`，預設埠 28761、需本機 token，不提供 CORS。原生 CLI 保留 framed TCP；兩種傳輸共用操作、權限與重複寫入 UUID 防護，不必把 CLI 改成 HTTP。
- 模組、原生 CLI 與 Skill 的 HTTP 操作均不需要 Node.js。Agent 可使用現有 PowerShell / curl，CLI 是可選工具。`npx skills add` 的 Node 需求只屬 skills.sh 安裝器；手動下載／複製 SKILL.md 也可安裝，沒有新增 npm 發布。
- [valheim-control Skill 頁](https://skills.sh/swear01/valheim-cli/valheim-control) 已能顯示實際 SKILL.md；乾淨安裝與原檔比對通過，同份 Skill 包含於社群 ZIP 與四種 CLI ZIP。個別 skills.sh 頁展示 SKILL.md，獨立 README 不會取代它；使用者決定暫不增加介紹或 skills.sh.json。這是本專案的展示決策，不是所有 Skill 的規則。

### SWOP 真實遊戲測試

Windows、Valheim 1.0.17 / network 40、BepInExPack 5.4.2351；由 v20 測試 profile 複製隔離環境，使用 CLITest1010 角色與 CLITest1011 世界。遊戲動作經 Skill 的 HTTP 客戶端呼叫原版方法／GUI 按鈕事件，沒有模擬 OS 鍵鼠；正式世界未進入，來源 profile DLL / cfg 雜湊未改變。

- 通過主選單、角色／世界選擇、GUI 按鈕發現、過期 UI ID 與遮擋拒絕；500 ms 移動 lease 到期、實際位移 1.673 m、轉向 30°、背包及移動閘門、直接 console help、玩家查詢、緊急停止。
- 通過背景持續執行、手動暫停時拒絕輸入及繼續、GUI 儲存／登出／退出；HTTP 與 CLI 重複寫入 UUID 跨傳輸拒絕。
- 100 次循序讀取全通過，中位 13 ms、p95 17 ms、最大 19 ms；8 次並發讀取全通過。這是同機短測讀取延遲，不代表 Agent 整體延遲。
- 校準過的桌面觀察器於完整啟動至退出約 2 分 40 秒內取得 10,265 筆樣本：前景變更、游標限制變更、新可見 console 視窗皆 0。BepInEx Error / Exception 行數 0；退出後程序與 listener 皆已停止，臨時觀察器／排程已移除。
- 世界載入時有 4 次唯讀查詢因 2 秒主執行緒佇列期限回覆「Expired before execution; action cancelled」，載入完成後恢復；等待 ready 再重試唯讀。不確定是否執行的寫入不可盲目重送。第一次測試 harness 25 秒期限短於 launcher 的 30 秒等待，調為 60 秒後完整重測，非模組故障。

未驗證：多人／dedicated server、長時間耐久、全部戰鬥／建築／農作動作、全部第三方 GUI、Linux / macOS 真實遊戲、一般桌面啟動或所有輸入裝置。此次也沒有重建 QoL 模組包，不得據此宣稱最新包版本或完整自主遊玩。

社群 ZIP SHA256 `369cf0778a63325b5435638010048b7305519e6ca7eed0f529e0cbff681e38a6`；實測／公開 DLL SHA256 `2938893c136964609cd0dd857598bdd8a4393b69ac1855638d8f6e7be1c0d7e2`。

證據：`<project-root>/outputs/http-live-0.5.0/verification.json`、`outputs/skill-http-0.5.0/{verification,publication-verification,release-status-verification}.json` 與 `outputs/skill-http-0.5.0/public-readback/`。原始日誌、截圖、本機憑證與私人路徑不進公開記憶庫。

## 歷史：0.1.0，2026-10-10

以下是當時的 Node CLI 與功能限制，不代表目前原生 CLI 的需求或功能。

2026-10-10 在 SWOP 的 Windows、Gale 1.23.1、Valheim 1.0.17、Node 24.19.0 實測 GitHub `swear01/valheim-cli` 的 CLI / ValheimCliBridge 0.1.0。以 Gale 複製 `Valheim-QoL` v20 為 `Valheim-CLI-Test-v20`，新增獨立角色／世界 `CLITest1010`，未進入正式世界。CLI 從交付 tgz 本機安裝，沒有 npm 發布；測試未變更 CLI 原始碼。

- 真實 Unity / Mono 載入 41 plugins、0 skipped、0 failed。Gale Steam 啟動沒有拉起遊戲，改 Direct 成功，Direct 保留為目前選項。
- 兩輪各 60 次循序、8 次並發、1 次拒絕後恢復的 status，共 138 次成功；另測 players 與實際 Node CLI 程序。兩輪中位耗時 2.63 / 3.69 ms，最大 19.91 / 29.67 ms，僅代表同機短測。
- 錯誤憑證、未知操作、AllowTeleport=false 的寫入被拒絕，隨後查詢正常。
- 透過 F10 / Mods settings → Valheim CLI Bridge 開啟 AllowTeleport，從距先前安全落地位置約 2.97 公尺處傳回；started 之後查 status，確認 teleporting=false、回報座標誤差 0，記錄約 0.93 秒。同一寫入 UUID 再次提交被拒絕。
- AllowTeleport 可由遊戲內 Configuration Manager 即時切換；Bridge Enabled / Port 必須重啟。最後 AllowTeleport=false，登出至主選單，僅監聽 127.0.0.1:28761；臨時 GUI 排程工作已移除。
- 0.1.0 僅有 status、目前載入的玩家清單、主機自身傳送；沒有挖礦／建築／戰鬥或任意 console 執行。加入端即使 admin 也不允許寫入，尚未實測該情境。

未驗證：第二玩家、多人地圖同步、遠距或未知地形傳送、長時間耐久與記憶體洩漏。不可把此次通過描述成完整 agent 遊玩能力或全面相容保證。

## 繼續測試

測試 profile 在 `%APPDATA%\com.kesomannen.gale\valheim\profiles\Valheim-CLI-Test-v20`；CLI 在 `%LOCALAPPDATA%\ValheimCliTest\cli\node_modules\.bin\valheim.cmd`。token 只留在 profile，不得打包或公開。測試報告放於 `<project-root>/outputs/ValheimCLI-SWOP-實機測試-20261010.{md,json}`；原始報告含私人機器資訊，公開前另寫去識別摘要。

交付 tgz SHA256：`b257e5948a51b7650f98f38ed76cf6d54daec55ed1f9f96ed513fc36256e3c7f`；Bridge DLL SHA256：`859297b57d1566c5d3802d68c8154653523b8d59118331fa7b9d44e65c7ae552`。

## 公開交付 2026-10-10

GitHub PR #1 已合併到 main `7be7f419030ce2a04c1c6cf1525eb92bc94502d6`；最新 head 的 Gemini 無新增建議，Windows / Ubuntu CI 通過，合併後兩平台 CI 亦通過。本機重新驗證使用現有 .NET 9 執行檔與 DOTNET_ROOT；系統預設 .NET 10 無法直接執行 net9 fixture，非測試邏輯失敗。C# checks 與 5 個 Node tests 皆通過。

GitHub experimental prerelease `https://github.com/swear01/valheim-cli/releases/tag/v0.1.0` 已發布，同時提供原實測 CLI tgz 與 `ValheimCliBridge-0.1.0-Thunderstore.zip`，兩資產下載讀回 SHA256 相符。社群 ZIP 根目錄有 manifest.json / README.md / icon.png / CHANGELOG.md / LICENSE.txt，DLL 在 BepInEx/plugins/swear01-ValheimCliBridge/；僅依賴 denikson-BepInExPack_Valheim-5.4.2351，不含 token、設定、私人日誌或遊戲 DLL。ZIP SHA256 `0ac0c34a15a7fe4c32a3972b58bf498ba8978a3d0830d1192511414bf65a26e2`。

Thunderstore Valheim 社群最初上架 `https://thunderstore.io/c/valheim/p/swear01/ValheimCliBridge/`（現已 Deprecated，見下方更新），dependency string 為 `swear01-ValheimCliBridge-0.1.0`。本人登入後建立 swear01 Team（本人為 owner），官方 manifest validator 回報 No errors found!；上傳成功、社群列表版本／相依套件正確，新站未登入頁面也可讀取。從公開 Manual Download 下載 ZIP，SHA256 與原準備包相符，內部 DLL 雜湊仍為實測版本。分類 Mods / Tools / Client-side / AI Generated；網站明示 significant AI content 需標示，已遵循。新版網站登入狀態與 legacy 分開，實際透過已登入 old.thunderstore.io 上傳。CLI tgz 保留最初打包 README / runtimeVerified=false 建置資訊，更新實測範圍載於 release notes 與社群 ZIP README；npm registry 仍未發布。

## 公開名稱與圖示更新

依使用者要求，公開名稱改為 **ValheimCli**。管理 UI 沒有改名欄位，因此以新 package 上架 `https://thunderstore.io/c/valheim/p/swear01/ValheimCli/`，dependency string 為 `swear01-ValheimCli-0.1.0`；舊 ValheimCliBridge 在管理 UI 讀回狀態為 Deprecated，歷史下載保留。舊套件已安裝者需先移除再安裝新套件，避免重複 DLL。

imagegen 生成北歐奇幻 ValheimCli 圖示，原圖與 256×256 PNG 已留 workspace outputs 並上傳同一 GitHub v0.1.0 release。公開 ZIP 下載 SHA256 `dd1dbddc0ff0702cb0fdc465578c24fcf0617847aa17c394274cd37e4c66318d` 與本機包相符，icon.png 與生成的 256px 檔一致；內部 DLL SHA256 不變。plugin GUID / DLL / cfg / token 名稱保留 ValheimCliBridge，程式行為與 CLI 指令不變。本次沒有重跑遊戲，僅核對封裝、公開頁面、圖示與二進位雜湊。GitHub release 標題／說明已指向新名稱；舊 ZIP 以歷史資產保留。證據 `<project-root>/outputs/ValheimCli-0.1.0-發布驗證.json`。
