---
title: HexaPharma 極簡視覺方向與玩法審查交接
scope: project
project: hexapharma
status: active
updated: 2026-09-06
---

## 最新使用者方向

HexaPharma 的核心是「探索藥效地圖、發現配方、親手建成實體產線」：Big Pharma 的工廠 + Potion Craft 的地圖。使用者要求極簡、偏 pixel，但不強制 pixel art；避免泛黃光暈、過量亮邊和卡片網站感。這個偏好優先於舊 design 的 Orbital Wet-Lab／bone-white／amber 美術處方；不表示要換引擎或修改 sim 規則。

完整實作 brief 在 swear01/hexapharma issue #4。前端可與 #7 的所有已發現配方查閱一起做；兩者重疊 Game.tsx，注意 module ownership。

## 已驗證的審查教訓

基線 commit 19ff833c8fe48c13b84711996a98e52c73288bf0；未修復狀態請以各 issue 最新內容為準。

- #5：live production 直接使用累計 replayTicks 安全上限；100000 ticks 後拒絕下一 tick，Reset 只清 runtime，不能恢復。匯入防護預算不應變成整局遊戲壽命。修復不能單純取消 strict decoder／work bounds。
- #6：Production 每 80ms 派發 8 ticks；New Game confirmation 開啟時 timer 仍會修改背景 state。測 modal freeze 要先 Play，再跨多個 intervals 檢查，不能只在 paused state 測快捷鍵。
- #7：sim 保存多疾病 discoveredFormulas，但 UI 只查 currentDiscoveredFormula 的最後一筆。是查閱缺口，並非 save 丟失其他疾病配方。
- #8：正常預算的 reference/compiler 基準路線在五個 seeds 出現後段缺錢。seed14 完成前三病正利潤出貨後 cash655，Settle 要700。這不是所有手動解法無解的證明；需要完整 progression ledger，first-sale affordability 不足以驗證多疾病循環。

## 驗證界線

Reference/compiler 產線只用於白箱 balance 或視覺 fixture，不能當成人類 fresh-play 證据。自動 gate 證明規則／測試覆蓋，不證明探索可理解或有樂趣。玩家首次理解、失敗嘗試、最低現金、first-sale time 和是否想研究下一病仍需人工紀錄。


## 已合併的經濟修正與可重用教訓

PR #10（merge 7c892b4）將有限需求每次出貨的整數衰減由 9/10 改為 19/20，Settle 由 $700 改為 $670；成本結算時點、正常 $1000 預算與手動解謎不變。`npm run progression` 的 34 個 unique seeds（0..31、42、100）全部完成四份 authority shipping contracts、每病正 net，最低現金為 seed7 的 $103。完整 gate：688 unit tests、71 Playwright tests 通過。這仍是 dev-only 基準路線的存在性證據，不是所有玩家路線或真人樂趣保證。

- 只調低 patents 並讓原五 seeds 通過會過度貼合樣本；擴大樣本後仍有後段建造不可達。先量測完整 ledger，再調單一有限需求 pacing 參數，比把所有解鎖費壓到很低更有依據。
- ledger 的 `completed` 必須檢查真實 shippingContracts，不能只代表迴圈走完四病；CLI 對不可達、合約未完成、非正 net、低於容錯預算都必須非零退出。
- seed4 的 dev compiler 較早 BFS belt 會占用後面機器的 output exit，造成 `route start ... is occupied`。既有程式只預留 input approaches；一起預留 output exits 即可修復，不需要重寫 packing 或修改 public API。
- 經濟係數改動要一起檢查 README／active docs 和市場 E2E 的旧價格 fixture。此案將市場 E2E 的固定 fixture 更新為 22 次出貨，並驗證可見報價與 disabled reason；不要誤記成動態推導測試閾值。
- reviewer 要求把 optional chain 換成 non-null assertion 並聲稱可防 TypeError，是錯誤建議。`!` 會在編譯時消失；dev verifier 的明確缺失 outcome 診斷應保留。有效回饋與無效回饋分別處理，不為取得 pass 而削弱檢查。

## 前端回歸驗證教訓（PR #11）

- `cover` 放大固定 logical canvas 後，camera clamp 必須使用真正可見的裁切 viewport；否則畫面填滿了，但地圖邊界永遠停在畫面外。縮放 anchor 也要扣除置中的 crop offset。1280px 與 390px 的實際 map-corner E2E 可重現並驗證此問題。
- 桌面 Formulas 是可操作 world 旁的參考，手機則遮住整個 world。元件 active 與全域數字／Enter hotkeys 都必須使用同一個 visibility guard，不能只修 Factory 而漏掉 Research。背景 Production timer 與 world input 是不同條件；非 modal 配方查閱仍可生產。
- Escape 關閉 drawer 的處理要先於 focused input/select 的一般快捷鍵排除，同時保留 blocking dialog 的優先權。
- 正常產線也有 throughput bottleneck；不要使用 failure red 表示它。極小正 throughput 不應因固定小數位顯示成 0，native significant-digits formatter 足夠。
- UI 改標記、label 或入口位置時，掃受影響的 design/invariants/player-guide/playtest/UI contract；不要只改實作文件。完整 gate 已在程式提交 9b9a102 驗證 688 unit / 90 E2E；後續狀態以 PR 最新 head 為準。
- GitHub PR review comments 超過預設單頁後，必須 `gh api --paginate` 並依 `original_commit_id` 辨識新發現。不能只讀最後幾筆第一頁留言，或把 summary 的 Completed 當作 clean pass。

## 使用者已決定：完全開放存檔（2026-09-06）

使用者明確強調「存檔採完全開放，沒有什麼加密的必要，這很重要」。採可攜、可直接編輯的明文狀態存檔；不加密、不簽章、不綁瀏覽器、不防玩家修改合法資源數值，不要求收入來源證明或為改資源重算 checksum。保留結構、大小／工作量、安全整數與可執行狀態一致性檢查即可。Alpha 可直接換格式、破壞舊存檔，不做 migration 或 legacy 分支。#5 的存檔信任選擇已解除阻擋，不要再次詢問。已由 PR #12 合併完成，#5 已關閉。

## 開放快照的驗證教訓（PR #12）

- Save v11 直接冷還原有界狀態，移除終生 intent trace 與累積 replay 壽命；單次 batch 工作限制及整數表示上限仍保留。大量相同產品用相鄰 payload 分組，仍是可編輯 JSON，不是加密。
- 嚴格正常 Load 不代表禁止明確 Recover：同版本、大小及項數受限的外層資料可救回個別驗證通過的快照；非字串損壞項必須保留缺口，不能 filter 後接合跨損壞區段的歷史。讀取不覆寫原資料。
- 歷史鎖存的 deadlocked 旗標無法只用目前 tick 驗真；簡單試跑會誤拒合法週期 source。移除 permanent-stop early return，讓每個真實 tick 重新計算診斷，避免玩家編輯旗標永久凍結可運作產線。
- 最新成功 Research 與 discoveredFormulas 不僅要有對應 program，還要符合最新順序；重用 discoverFormulas 的冪等結果驗證，避免 UI 指向較舊配方。
- 既有 factory restore 已在 Int32Array 賦值前驗 unit ID 上限；review 指出 parser 缺同一 guard 不等於存在截斷漏洞。以保持質量守恆的超界 ID 存檔重現拒絕流程後再判定。
- 最終機械驗收通過 697 unit／90 E2E；最新 Swear 完整 review 為零 findings，CI 通過，合併後 main 建置與直接改資源載入繼續運行亦通過。
