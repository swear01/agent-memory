---
title: Vault Hunters Remastered small-server operations
scope: projects/dvlab-mis
status: active
updated: 2026-09-16
---

# Vault Hunters Remastered small-server operations

- 非正式、無獨顯主機上的朋友服使用官方 Server Pack 2.0.4（CurseForge file 8502840），客戶端對應 CurseForge 2.0.3（file 8502758）；兩者底層核心模組皆為 `the_vault-1.18.2-20.0.3-remastered.6872.jar`（SHA256: fc6adfeb... 一致）、Minecraft 1.18.2、Forge 40.3.11、Java 17，官方 ZIP 的 manifest 殘留 2.0.1 字樣。官方 Server Pack 的 `config/bcc-common.toml` 殘留 `modpackVersion = "2.0.2"`，導致 Better Compatibility Checker 判斷版本不合報紅叉，已手動修正為 `2.0.3` 與主流客戶端一致。升級前重新核對官方版本與 client/server 相容性；不要把舊版 Third Edition 當作同一模組包。
- 伺服器說明（MOTD）設為 `dvlab server`；伺服器圖示 `server-icon.png` 採用 DVLab 官方 Logo（64x64 RGBA）。
- 公開入口共用既有網站主機的公開位址，只轉發 Minecraft TCP 埠；沒有配置 Minecraft proxy，也沒有為新主機新增整機 1:1 NAT。域名、位址與路由規則的精確值以受限網路手冊為準，外網狀態查詢不等於真人登入驗證。
- 約四位朋友、不固定同時上線；槽位上限八人。保持正版驗證與白名單。Vault 戰鬥 Normal、死亡模式 NORMAL、研究隊伍加價啟用；關閉 PvP、一人睡覺可跳夜晚。不要自動由白名單建立研究隊伍。
- 戰利品目標約 1.25 倍：內建 `vaultLoot` 沒有此倍率，保持 NORMAL；16 份一般 wooden/gilded/living/ornate 數字等級寶箱 loot table，uniform roll 由 12–15 改成 15–19，平均為 34/27 ≈ 1.2593。這不是精確 1.25 或每箱固定增加 25%。Raw、Strongbox、Treasure、礦石、金幣堆與完成獎勵未改，物品權重與每次數量也未改。
- 對應路徑在遊戲目錄的 `config/the_vault/gen/1.0/loot_tables/`；改前保留原檔，驗證只有預期 roll 改變。升級可能覆蓋設定，不能對已修改值重複乘倍率。
- `the_vault reloadcfg gen` 可載入此設定；實測約 11 秒主執行緒停頓，安排無在線玩家時操作。日誌讀取 loot_tables 並成功 reload，只證明載入，不代表已做玩家開箱分布測試。
- 遊戲由 `vault-hunters.service` 執行，僅本機 FIFO 管理控制台，不開 RCON。每日臺灣時間 04:25 開始 5 分鐘、1 分鐘、10 秒遊戲聊天通知，04:30 正常停服備份後重啟；這不是偵測 Vault 後延後維護。
- 本機與 NAS 各保留 7 份 daily、4 份 weekly；NAS 缺失時保留本機備份並重啟遊戲，但備份工作回報失敗。完成真人登入、拒絕非白名單、Vault 體驗與多人負載，仍需另外驗收。
- 主機的共享家目錄正式切換及整機重開機是另一項尚未完成的工作。不能因 Minecraft 或備份成功，就把 NIS/NFS 完整重開機驗證標成完成。
- 正式文件為獨立的 Minecraft 維運 runbook；帳密／硬體回到 Overview、IP/MAC/NAT 回到 network manual、主機 NIS/NFS 實作回到 New Server Setup。
