---
title: Valheim CLI Bridge 0.1.0 SWOP runtime verification
scope: projects/valheim-cli
status: verified-with-limits
updated: 2026-10-10
---

# SWOP 實機結果與限制

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

Thunderstore 上架仍待本人登入與 Team；不可把 GitHub prerelease 或已備妥 ZIP 描述成 Thunderstore 已上架。官方 manifest validator 也要求登入 Team，只有本機 ZIP 結構／內容檢查完成。CLI tgz 保留最初打包 README / runtimeVerified=false 建置資訊，更新實測範圍載於 release notes 與社群 ZIP README；npm registry 仍未發布。
