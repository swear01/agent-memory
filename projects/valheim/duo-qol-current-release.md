---
title: "Valheim Duo QoL v52：ComfortTweaks J 鍵與目前模組包設定"
scope: projects/valheim
status: verified-packaging-runtime-unverified
updated: 2026-10-16
---

截至 2026-10-16 的正式交付為 v52，取代此筆先前的 v41 狀態。續作先核對 repository、lock 與最新交付，勿把舊快照當成當前版本。

- 統一工作目錄為 `<user-documents>/valheim-duo-qol`；公開 repo 為 `swear01/valheim-duo-qol`。研究、交付、CLI、私人設定分別在 `research/`、`deliveries/`、`cli/`、`private/`，不提交私人內容。舊 Codex 路徑可能只是 symlink。
- v52：`Valheim-Duo-QoL-v52.r2z`，profile `Valheim-Duo-QoL-20261016-v52`，50 個線上套件、40 份公開設定、41 份雙人私人設定。交付在 `<project-root>/deliveries/2026-10-16/`，目前聊天的 `outputs/` 為 symlink。
- ComfortTweaks 3.3.11 使用 Hexium；`xyz.alcan.comfortcalc.cfg` 的 `2. Resting / Comfort Range = 50`、`5. Comfort Pieces List / Show Comfort Pieces List = true`、`Toggle Visibility = J`。其餘規則使用模組預設，沒有改火源保暖半徑。清單首次進入世界時隱藏，按 J 顯示；列出提供舒適度與已解鎖的缺少家具。
- 避免使用 ComfortTweaks 預設 F6：鋤頭取樣高度與 PerfectPlacement 預設對齊功能也使用它。v51 曾設 Home，但 MovingDay 的預設 `HeightResetHotkey` 也是 Home，搬移時會重設高度；v52 改 J。檢查衝突須同時看 CFG 覆寫與 DLL 的未覆寫預設，不能只搜尋精簡 CFG。J 在核對的包內／原版與相關建築預設未發現占用，尚未遊戲內實測。
- AzuCraftyBoxes 取料半徑 50 公尺；單一箱子是否參與製作為 RightShift。PlantEasily 種植範圍修飾鍵為 RightShift，採收仍 LeftShift。Infinity Hammer 左右排列 RightAlt＋滾輪，前後排列 RightShift＋滾輪，縮放為 None；準星對著可操作箱子時按 RightShift 仍可能同時切換該箱子的取料狀態。
- PlanBuild 自 v47 移除；保留 Infinity Hammer、World Edit Commands、MovingDay。ValheimTerrainTools 的 Removal / MaximumRadius 由 0.5 改 10，解除最小＝最大造成滾輪鎖住；一般 Brush 上限仍 10。Removal 上限同時影響 Remove builds，無材料返還或復原。
- v43 起內附 SlashBack Return Fix 1.0.0，原 SlashBack 1.0.2 保留；取消使用冷卻與死亡後 120 秒期限，保留 5 秒站立倒數、移動取消與最新一次死亡位置。
- 私人交付固定執行 `python3 -m scripts.build_private_package`，沿用本機 `<user-home>/.config/valheim-duo-qol/private-admins.json` 的兩位管理員（設定檔權限 0600）；缺少設定即停止，不能改交付公開包。私人名單、permissions.yaml 與私人 ZIP 不公開，也不寫入共享記憶；此筆只保存包的 fingerprint。私人包在 `dist/private/`，加入權限檔並開啟 `Automatic devcommands`；公開包在 `dist/`，Actions 只交付公開包。換機須安全搬移私人設定，GitHub／共享記憶不提供名單備份。

主分支僅保留最新 `history/v52/`；舊版 archive／lock 留在對應 tags，不改寫舊版。兩人退出遊戲並備份角色／世界後，在 Gale 匯入同一份私人包為新 profile，勾選 Import all files。

v52 公開 SHA256：`2ea7d34bf4bb713110c47693b670eb6c47e0940dfa6a879a84eaf6ae953dcd0c`；私人 SHA256：`6ed1c2927c97a5feb7f334a6183de49054ff6ed1078d2d8eb5575a811bc203e9`。公開／私人同名，必須依來源及雜湊區分。

PR #21 已合併；審查 head 與 v52 tag 為 `8b5669c8471a27c655ff2873a52f2eea816e8214`，目前 main 為 `9603818a29fce43160e4f2c8763c87011477470b`。15 項 Python 測試、CI 27 項 SlashBack 檢查、52 版 archive／lock／tag／profile、公開／私人確定性重建通過；CI 產物與本機公開包逐位元組相同，兩位私人管理員政策保留。Codex 對同 head 的 code/security 審查完成、無 inline findings 並有正面反應；其他 pending reviewer 不當成通過。任務 worktree／分支已清理，local／remote main 與 tag 已核對。

未修改或重開既有 Gale profile／遊戲／角色／世界；Gale GUI 匯入、J 鍵、50 公尺邊界、UI／效能、Unity／Harmony 掛接與雙人行為仍未實測。證據為 `<project-root>/docs/comfort.md`、`lock.json`、`deliveries/2026-10-16/Valheim-v52-驗證.json` 與 GitHub PR #21；同步與 QMD 的結果須另核對 Git／索引讀回。
