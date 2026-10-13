---
title: "Valheim Duo QoL v41：正式交付、私人雙人權限與續作方式"
scope: projects/valheim
status: verified-packaging-runtime-unverified
updated: 2026-10-13
---

# 目前版本與來源

使用者指定直接命名為 **v41**，不使用 `v40-private-admins` 或私人版尾碼。至 2026-10-13，交付為 `Valheim-Duo-QoL-v41.r2z`，Gale profile 為 `Valheim-Duo-QoL-20261013-v41`；50 個線上套件、41 份設定，含兩人管理員。續作先讀目前 repository、最新 lock 與交付驗證，不退回 V27／V32 或 CLI 工作目錄的舊包。

- 交付目錄：macOS `<codex-root>/2026-10-13/new-chat/outputs/`。包、`Valheim-Duo-QoL-lock-v41.json`、`Valheim-Duo-QoL-v41-驗證.json`、`Valheim-雙人模組包-安裝指南-v41.html` 與 `Valheim-v41-Git同步紀錄.json` 一起核對。
- 私人交付 SHA-256：`ca81f48f6db2794bb7f9caee5d59172dcc3ea06d01c24b9340bd453fb46a7f4c`；24,780 bytes。
- 公開 repo：`swear01/valheim-duo-qol`；本機 checkout 為 `<codex-root>/2026-10-12/new-chat-4/work/valheim-duo-qol`。公開 v41 為 50 套件、40 設定，不含玩家 ID；同名公開 ZIP 的 SHA-256 為 `a1963ca48a1d356d9c6f954196a75fd8be8b957ecbb263d56d04bd0d4d7603e6`，不可當作已含雙人權限的交付。
- PR #10 已合併；v41 tag／審查 head 為 `7edac1d48110daa39fb38b08719ef39fabc74621`，當時本機與遠端 main 為 `f29b023e4c68a8b528ce3f22bfae79d68dc9a31f`。任務分支與乾淨 worktree 已移除。

# 私人設定持續沿用

固定來源是本機 `<user-home>/.config/valheim-duo-qol/private-admins.json`，權限 0600；內含已核對的兩個 Steam 帳號，**不要把原始 ID、含 ID 的 lock、私人封包或權限檔寫進公開 repo／共用記憶／Actions artifacts**。使用者的同學開房，使用者加入；房主載入的權限檔控制加入者授權。原生 Steam 快取與公開 Steam profile XML 已比對，但連線時實際 ID／授權仍需遊戲驗證。

在 repository 根目錄執行 `python3 -m scripts.build_private_package`。先核對並重建公開版本，再沿用固定私人設定新增 `BepInEx/config/permissions.yaml`、開啟 `Automatic devcommands`；兩個不同帳號設為管理員，Everyone 的 permissions 指令預設禁止。缺少或無效私人設定即失敗，不能退回公開包交付。

私人產物在 `dist/private/Valheim-Duo-QoL-v版本.r2z`；公開包在 `dist/`。檔名及 profile 都沿用正式版本；公開 Actions 只上傳 `dist/*.r2z`。私人名單目前只有本機保存，未設定雲端備份；換機需安全搬移私人 JSON，公開 GitHub 與共用記憶無法還原名單。不要誤稱玩家 ID 已隨此記憶跨機同步。

兩人退出遊戲並備份角色與世界後，在 Gale 匯入同一份私人完整包為新 profile，勾選 Import all files；若已有同名 profile，先備份並另取名稱，避免覆蓋。沒有改動、重開或安裝到目前遊戲／profile／世界。

# 既有功能與生存設定

v41 沿用 v40 的全部套件 pins、原有 CFG 與內附 DLL；私人包只另加權限檔並啟用自動 devcommands。AzuHoverStats 1.1.11、MovingDay 1.1.0、HUDCompass 1.2.1、Infinity Hammer 1.90.0、World Edit Commands 1.81.0、PlanBuild 0.20.0、Server devcommands 1.115.0 均已在 lock 核對。

- HUDCompass 的 `Show dynamic pins on map = true`；MovingDay 的 `TerrainModifierKey = LeftControl`，地形搬移使用修飾鍵。
- Infinity Hammer：`Group resource cost = true`；`No resource cost`、`No stamina cost`、`No durability cost`、`Ignore no build`、`Ignore other restrictions`、`Ignore wards` 全為 false，保留支撐與消耗。水平排列為右 Alt＋滾輪，F4 凍結。
- PlanBuild 先規劃、投料建成，J 鏡像；管理員按 Ctrl 放置仍可能切換免費直接建造，不可宣稱所有管理員操作都強制消耗材料。自動飛行、無敵、隱形及免費建造未開啟。

# 驗證邊界與歷史

15 項單元測試、41 版原始 archive／lock／tag／profile 快照、公開與私人逐位元組重建、精確兩帳號 YAML、公開 tree／diff／包的真實 ID 掃描均通過。PR CI run `38357557473` 成功，下載 artifact `11793226998` 與公開 v41 一致且無私人權限；GitHub v41 tag tarball 與本機 archive／lock／profile／builder 一致。Gemini 在該 head 明確沒有審查意見；確認 CLEAN 後以 head guard 合併，並從已同步 main 重建相同私人交付。

尚未完成 Gale GUI 匯入、遊戲內雙人授權、支撐／消耗／藍圖行為及長時間相容性實測；封包驗證不代表遊戲測試。主分支只保留最新 `history/v41/`，舊版 archive／lock 留在不可改寫的舊 tags；不要把私人 ID 上傳到舊 tag 或以永久刪除代替可回復清理。
