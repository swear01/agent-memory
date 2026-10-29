---
title: Grok Bot GIF 頭像的原生設定入口與 Mac 播放驗證
scope: tools/grok-bot
status: verified
created: 2026-10-09
updated: 2026-10-30
tags:
  - grok-bot
  - gif
  - avatar
  - macos
---

# Grok Bot GIF 頭像

## 已驗證的方法

2026-10-09 在 Mac Grok Bot 0.68.1 完成五個自訂 GIF 頭像設定，並確認 App 實際播放。

1. 使用已完成的循環 GIF；本次檔案約 1.1–1.3 MB，512×512、10 畫格、無限循環。
2. 請目標 Bot 將 Mac 原始 GIF 原封不動複製到它的雲端 `/workspace`。
3. 請 Bot 用內建的「檔案路徑設定自己頭像」功能設定該雲端 GIF，不經桌面裁切編輯器、不轉 PNG、不重新生圖。
4. 比對原檔與保存內容的 SHA-256，檢查 GIF 格式、畫格數與循環設定，再確認 Mac App 的實際動畫。

可用指示：「請把 `/workspace/avatar.gif` 原封不動用內建功能設為自己的頭像，不裁切、不轉 PNG，核對保存後格式、畫格數與 SHA-256。」

本次直接提供 Mac 檔案路徑首次回報 `Could not read`；先複製到雲端再設定成功。這是已重現的可用流程，不代表所有版本都無法直接讀 Mac 路徑。無須安裝生圖技能、修改 App、改資料庫或呼叫未公開 API。

## 桌面裁切入口為何變靜態

Mac 0.68.1 的頭像編輯器保存時將裁切內容畫到 256×256 canvas，執行 `toDataURL("image/png")`，透過 `setAgentAvatarBytes` 保存 PNG。即使候選預覽顯示 `.gif`，且確實按下「設定頭像」，保存結果仍可能只有單張 PNG。

原生選檔曾未載入 GIF 候選；Finder 複製 GIF 後在上傳區貼上可載入，但仍不能繞過編輯器保存時轉 PNG。未查明選檔未載入的根本原因，不把它推定為 GIF 不受支援。

Bot 內建入口保存的檔名可能叫 `avatar.png`，內容卻仍為 `GIF89a`；必須看實際位元組與解碼結果，不能用副檔名判定格式。Mac 的 `roster-avatars` 快取可保存原始 GIF 並播放。

## 完成證據與邊界

- 五個 Mac 頭像快取皆與對應原始 GIF 的 SHA-256 完全相同；GIF、512×512、10 畫格、loop=0。
- 擷取實際 App 畫面 85 次，歷時 7.548 秒；每個側邊欄頭像均捕捉到九個不同姿勢，並目視核對。
- 原始檔與證據保留於 `<task-root>/outputs/grok-action-avatars/`：`native-import-verification.json`、`native-five-app-playback.gif`、`native-five-motion-proof.png`、`native-avatar-research-2026-10-09.md`。
- 先前桌面編輯器轉 PNG 的紀錄只是該入口的限制，不能泛稱 Grok Bot 無法使用動態 GIF。
- 這些是固定循環 GIF，沒有接上思考、工作或閒置狀態切換；原生造型的狀態動畫能力不能推定適用自訂 GIF。
- 未看到同學本人操作；只核對公開作者的 Bot 自行設定方法並自行重現。未測其他 App 版本或手機端。

## 後續驗證原則

工具回報設定成功、頭像可見或檔名為 GIF，都不足以單獨證明動畫保存。先核對保存位元組，再對實際 App 連續取樣至少覆蓋最長循環；單張畫面或短時間恰好落在休息姿勢，不能據此判定靜態。

## 2026-10-29：讀書鯨與教材交接

Mac Grok Bot 0.78.1 再次驗證上述原生設定方法。新增的「讀書鯨」沿用同系列大頭、極小身體的鯨魚娘風格，採粉紅配色；GIF 為 512×512、16 個不同畫格、2.8 秒、無限循環，動作依序是從左邊接書、翻兩頁、往右拋出。圓形頭像內保留臉、鯨魚髮飾與書本。

- 透過 App「新 Bot」建立角色，再在新對話傳入教材、教學方式與已讀進度的摘要交接。這不是完整歷史對話匯入。
- 通用讀書角色以 LSV 為首項任務；沿講義與逐字稿順序，用繁體中文精簡教學，一次一個概念。公理證明逐行列等式與理由，不用真值表或 evaluation 代替。
- 本次接在第 3 週第 1 段末尾，進入第 2 段 Boole 展開與可表示性；交接內容不可假定學習者已掌握展開定理。Bot 已提出一個接續問題並等待回答。
- Bot 回報：雲端交接資料夾 14 個檔案的 SHA-256 均符合 Mac 原檔，其中 12 份教材符合 `SOURCE-MANIFEST.json`；角色、教法、教材位置與進度已寫入它自己的持久工作記憶。這些是 Bot 的回報，沒有另行獨立讀回雲端檔案或記憶檔。
- 獨立核對 Mac `roster-avatars` 保存位元組，SHA-256 與原始 GIF 完全相同；保存檔雖叫 `avatar.png`，內容仍是原始 16 格 GIF。
- 對實際 App 側邊欄的新頭像擷取 20 次，涵蓋 7.137 秒；解碼 RGBA 後有 11 個不同姿勢，並目視核對翻書、表情與書本移開。這證明 Mac 0.78.1 的實際播放，未測其他裝置，也未接上工作狀態切換。
- 成品及驗證保留於 `<task-root>/outputs/study-whale-2026-10-29/`：`study-whale-16.gif`、`delivery.json`、`native-bot-readback.json`、`playback/samples.json` 與頭像取樣。

### 本次工具限制

`grok-bot-cli` 0.12.4 的 `doctor` 找不到 App 的 `gateway-descriptor.json`，因此該 CLI 沒有可用的 gateway session；不能把已安裝 CLI 或 App 已登入視為 API 已連通。此次直接使用原生介面完成，無須處理 Keychain 或修改 App。

背景 AX 掃描可讀到介面，但單靠 `AXPress` 回報成功並未讓頁面改變；明確啟用 Grok Bot 後，按新取得的 AX 邊界輸入及點擊才完成建立與傳送。每次動作後要重讀目標角色標題和實際頁面，不能以按鈕呼叫成功推定建立或送達，也不要在未確認前重複建立 Bot。


## 2026-10-30：七款 GIF 的首尾循環修正

後續更正：本節只證明格式、保存與實際播放。使用者隨後指出 Mail Bot、HAPI、鸚鵡 Bot、NTU COOL 的頭部及手部仍會不自然抖動，拒絕這四款的動作品質；最新修正見下節。

使用者語音中的「CSS」是「十六張」的辨識錯誤，已明確確認要 16 張循環 GIF，不是 CSS 動畫。保留既有大頭、極小身體的造型。

- 讀書鯨保留原 16 張畫稿，改為 `0..15,14..1` 的 30 時間格往返播放；每格 120 ms、總長 3.6 秒、不重複端點，移除原首尾各 300 ms 停頓。逐格解碼像素與舊 GIF 對應畫格完全一致；動作會反向返回，並非重畫完整接書／丟書閉合序列。
- Grok Bot、Mail Bot（舊稱 Gmail Bot）、NTU COOL、HAPI、電腦操作、鸚鵡 Bot 各重新生成 16 張畫稿；512×512、每格 120 ms、1.92 秒、無限循環。生成 sheet 實際為 1254×1254，分格後上採樣至 512，不能稱原生 512 畫格。
- 生成提示應指定完整週期與固定鏡頭、比例、背景，讓第 16 格是接回第 1 格前的姿勢。驗證要包含最後到第一格的變化、一般相鄰格比較和端點時長；畫格數、不同 hash、loop=0 或 prompt 寫了「首尾相接」都不能單獨證明循環順暢。生成畫稿仍有逐格位置漂移，RMSE 比較不等於補間或完全平滑保證。
- 七隻都透過各 Bot 內建頭像功能更新；獨立核對 live avatarVersion、Mac cache 完整 SHA-256 與 GIF 解碼規格，7/7 匹配。快取可能晚於 Bot 回覆落地，不要在 cache 尚未出現時立刻當作設定失敗。
- Mac Grok Bot 0.78.1 實際視窗連拍 28 次、跨 10.723 秒，六款新圖各捕捉到 16 種姿勢，讀書鯨捕捉到 10 種；證明七隻在 Mac 實際播放，未測其他裝置，也未接工作狀態動畫。
- 本次 AX 僅有選單、沒有視窗內容；以原生視窗截圖確認目標角色後輸入。Swift 中呼叫 `NSRunningApplication.activate` 後立即結束程序，啟用可能未完成；同程序保留 `RunLoop.current.run(until:)` 後才確認前景並操作。啟用回傳成功、按鍵送出都不能代替畫面或送達驗證。

成品與證據位於 `<task-root>/outputs/seamless-avatars-2026-10-30/`；保留原檔。`delivery/cache-verification.json` 記錄保存位元組核對，`delivery/native-playback-verification.json` 與 `delivery/native-motion-proof.png` 是實際 App 播放證據，`delivery/summary.json` 記錄限制。生成與首尾差異資料在各角色子目錄；不要把來源 GIF 預覽當作 App 播放證據。


## 2026-10-30：原畫局部動作取代逐格重畫

使用者明確更正：重點是頭部不要莫名亂擺，不是把手與道具固定、再讓頭擺動。Mail Bot 應把信封往前推再收回；NTU COOL 應揮同一隻手；鸚鵡的麥克風與持握手要一起動。逐格生成會改變頭部比例、手掌及道具形狀；把這些畫稿往返播放，或只比較首尾與內部最大 RMSE，都無法消除中途抖動。

- 回用先前接受的單張原畫，以固定的局部材質變換連續取樣：Mail 信封與雙手 `1→1.12→1` 前推／收回；NTU 固定原本張掌繞手腕小幅揮動；鸚鵡固定握法、麥克風微抬返回。三者頭臉固定。HAPI 只做單次 `0→0.8°→0` 歪頭，領口與披肩固定。每款 40 時間格、60 ms／格、2.4 秒，往返路徑重複姿勢是預期行為。
- 原始 PNG 未改動；raw 第 0 格與來源像素相同，第 40 相位精確回到第 0 格。獨立確認所有解碼 GIF 格的指定頭臉區域不變，NTU 須特別保護與手掌相鄰的臉頰／眼鏡，不能讓局部遮罩溢入臉。HAPI 的固定檢查區域是披肩。
- 共用色盤後再做 GIF 差異矩形壓縮，須完整解碼比對 RGBA 與時長；本次 ImageMagick 優化改變解碼色彩，未採用，改用保留原索引色的無損差異矩形編碼。
- 四款已更新，live avatarVersion、完整 cache SHA-256 及 GIF 規格 4/4 匹配；讀書鯨、Grok Bot、電腦操作的版本與 cache hash 不變。Mac 0.78.1 實際連拍 28 次、10.461 秒，四款均有動作；Mail、NTU、鸚鵡的頭部取樣區域各只出現一種像素 hash，未觀察到頭部跳動。
- 成品、renderer、原畫來源與驗證保留於 `<task-root>/outputs/stable-avatars-2026-10-30/`。`delivery/rig-verification-independent.json` 是局部固定與閉合檢查；`delivery/cache-verification.json` 與 `delivery/native-playback-verification.json` 是保存與實際播放證據。尚未取得使用者對本次修正版的主觀驗收；未測其他裝置。這是局部影像變形，不是 3D 骨架；鸚鵡沒有嘴型同步，HAPI 未新增眨眼。
