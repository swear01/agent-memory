---
title: Minutescape：OpenSpeedy 的兩種影格時鐘與 Construct 3 步進修正
scope: tools/openspeedy
status: active
updated: 2026-11-18
---

Steam Minutescape（App 3327170，build 25433331，NW.js 0.100.1／Construct 3）搭配 OpenSpeedy 3.3.20。已驗證 stock package.nw SHA-256 BE9F864E19141B337B2F43CD1E4F54A7632BACDA5BB310E86850A986AAF43119。DLL 載入和 OpenSpeedy.<pid> mapping byte 1 只證明原生加速開啟；到 OpenSpeedy 安裝目錄執行 bridge64.exe GETSPEED 的 stdout 可確認共享倍率，不能代替玩法量測。CLI ISENABLED／STATUS 不是可靠的程序啟用查詢。

## 計時根因及實測

安裝版 package.nw 的 data.json project[37] 是 vsync。scripts/c3runtime.js 將 requestAnimationFrame (rAF) 的 timestamp 送給 Tick 和 _MeasureDt。非影片匯出模式把超過 0.5 秒的差值歸零，其餘夾在 0 與 1/30 秒之間，再乘遊戲的 _timeScale。Dodge 事件 #34 把 System.Exps.dt 累加到 levelTime，#82 存入 recordTime。遊戲本身的閃避慢動作及子彈碰撞需要保留，不能只把 dt 上限放大。

- renderer-only：獨立 NW.js clock probe 用外部 Stopwatch 驗證 10× 時 rAF 仍是約 1×，performance.now()／Date.now() 約 10×；實際完整隔離 Dodge 也證實 stock levelTime 約 1×。初版相容排程用 performance.now() 加短步，在這條路徑得到 9.994×。
- 六程序 NAME：正式 Steam 試用有進 Dodge 後明顯卡頓；隔離完整遊戲重現停在接近首幀。兩個時鐘每回呼的中位差值都約 156 ms，而初版相容排程錯把原始 rAF 差值大於 125 ms 當成失焦中斷，持續歸零。保持隔離 fixture 其餘五程序不變，只停用主程序，rAF 立即回到 16.7 ms，關卡恢復。主程序使用 in-process GPU；未有證據把 Minutescape 的這次卡頓歸因於 Steam 原生等待。單一隱藏 stock fixture 在每真實秒約 539 影格的另一場景也量到約 9.44×；不能把 stock 有效倍率當成固定。
- OpenSpeedy NAME 每秒重套同名程序。直接對各 PID 執行 DISABLE 後，保存的 NAME 規則會重新啟用，這是設計行為。單以 OpenSpeedy 設定倍率作為遊戲控制（不使用原生注入）雖可行，但使用者明確要求由真實 OpenSpeedy 的程序開關套用，不能拿純倍率模式交付。

## 實裝

最終直接套用補丁只修改 c3runtime.js：在 Dodge 對已變速的原生時間差做序列 Tick，每步不超過 25 ms、每幀最多 24 步且盡量在 8 ms 真實時間的工作預算內，最後只繪製一次。分辨 renderer-only 與兩個 clock 同加速；有可用的 WebAudio currentTime 時用它驗證真實間隔與預算。用原始倍率時間一次，不再乘全域值。保留原 _timeScale、原版 delta cap、關卡切換與暫停時的中斷。超出預算的時間不排進積壓。

最終 SHA-256 29E2993AD2166413A58C905961B727955CE8DDADB73A40FFDBBEF7312B572593。無 Steam 整合的 NW.js 完整隔離遊戲，在約 63–64 個真實 gameplay callback／秒時，六程序實際注入啟用、關閉、再啟用皆成功。10× 兩輪有效 levelTime 量到 7.177×／8.122×，關閉時 1.000×；未停住或出現未處理 JS 錯誤。不要宣稱正式存檔或重場景已量到完整 10×。正式 Steam 重開驗證：載入相同修改 runtime、六程序均實際注入啟用、NAME 恢復、設定仍是 10×，視窗回應。只有 package.nw 被修改，保留原版備份；未修改存檔。


## 遊戲內沿邊巡走

同一 build 新增巡走時，只攔 Dodge 的 Player Movement 事件 #38（SID 944808599577142）的三個原生手動移動子事件；其他事件、Dodge 啟動與死亡照常。spr_player SID 971355649700802 實例使用原生 MoveTo 速度和 Tick，先向最近邊投影，再順時針走四角。場地不一定是 layout 的 1920×1080：實測初始 spr_box 牆為 200×200；以 spr_box SID 253185647818588 的 left、right、up、down 活實例中心位置計算可走矩形，再加半個角色寬高、像素邊距。這比讀固定畫面座標或無箱時都取同一矩形可靠。F8 只改本次程式的模式偏好，關閉時清掉巡走 waypoint，再放回原生滑鼠／手把事件；不開始、重開或編輯存檔。

隔離完整遊戲經 Main Menu 到 Dodge 的 test-only 轉場驗證全四邊、F8 關閉後滑鼠可移到中心、再開啟、離開關卡停止；全圈測試為避隨機死亡只在隔離 fixture 停用碰撞，正式補丁保留碰撞。安裝仍只改 c3runtime.js（package.nw），SHA-256 C759BB370C0A43FA1D742409F6EFA571D4E3D1946E4A992B49A699BCF9E8BDE6。備份 package.nw.before-patrol 是原本的直接套用 OpenSpeedy 版本；測試還原不移除 OpenSpeedy 修正。Steam 重開已確認載入同一修正版與原有六程序 OpenSpeedy NAME 掛鉤，沒有寫入倍率或存檔。
同一隔離版另以 OpenSpeedy 當時的 25× 設定、六程序原生注入，短樣本實測關卡約 19.126× 且連續巡走至少三圈；未變更共享倍率。

## 100× 原生隔離測試、巡邊及天賦樹

- OpenSpeedy v3.3.20 的倍率是 `speedpatch64.dll` 的共享 PE 資料段；`OpenSpeedy.<pid>` mapping 是啟用位元。要量不同倍率而不碰正式遊戲，可以實體複製官方 `bridge64.exe` 和 DLL 至隔離目錄，保持同一份複本 DLL 映射，使用同目錄的 bridge 只注入已核對路徑的 fixture PID。核對每個 fixture 的 DLL 完整路徑和 flag；同時驗證官方裝置的 GETSPEED 沒變。實測複本 100×、正式 25×，兩者獨立；不用改 DLL 或正式 GUI。
- 舊排程 600 ms 分類上限和 24×25 ms 步進在 100×、60 個 callback/s 時無法接收約 1667 ms 的正常加速間隔。新版放寬原始時鐘範圍、有可用 WebAudio 時優先用它判斷，100× 高速用原版 1/30 秒 dt 上限和約 11 ms 真實工作預算，不讓補算積壓。實際 100× 六程序測試約 29× 有效 Dodge 計時、63–64 個畫面 callback/s；每個完整 Tick 約 0.9 ms，事件/碰撞階段花了約八成。不能拿 GUI 設定說已實跑滿 100×，也不能把隱藏視窗高頻 callback 與固定 60Hz 的數據直接比較。
- 升級介面 #369 / #370（SID 226392173888185、874634095954579）原用 Touch.SpeedAt(0)×dt 算右鍵拖曳。加速使兩者時鐘比例失配。改用 Mouse canvas CSS 座標的相鄰位移，在同一畫面轉換至世界座標；僅替換 #370 的 Scroll 動作。完整隔離版 25× 同一掃動距離在 zoom=.8 為約 768 layout px，與不掛鉤一致；100× 亦一致，原滾輪仍生效。
- 巡邊仍每 Tick 讀 spr_box 活實例。重算新矩形後要先檢查角色到所走邊的垂直距離，大於 8 px 就沿原側重新貼回、進入 4 px 內才再朝角落走。否則即使牆座標更新了，角色仍會斜切向新轉角。真實 100× 注入的隔離副本放大牆 120 px 已驗證回同一邊後繼續轉角。

F9 自動刷分使用原始 UI 事件：Main Menu Play（有存檔走原版讀檔，否則等 Startup 提示）、Dodge 死亡結算完成後 OnAnyClick 進 Upgrade、等 Upgrade 的原生 IO / 存檔完成再送原版 Enter 並傳回 Dodge。不要直接切 layout 或寫分數。Construct 事件變數需要 `GetTypedValue()` 讀 boolean，`GetValue()` 讀到的是 0／1。獨立實機存檔在正常碰撞下累積紅分到 36，重開載回相同存檔後繼續增加到 39。F9 初始關、開啟時自動打開沿邊巡走；F8 可手動控制沿邊巡走。

100× 進一步取樣顯示 `[M] Background Effects`（群組 SID 799488624751281，每步約 .32 ms）是最大的單一耗時項；只讀寫背景粒子類別 94。在高倍速批次的首個步驟跑該群組，累積額外步驟的遊戲 dt 給下個畫面的背景移動與淡化，並在這個群組執行後還原原 dt。粒子生成與亂數取樣頻率會改變；計分、技能、彈幕及碰撞事件仍照算。實體隔離 native 100×、15 ms 真實影格間隔下限下，最終 42.14× 實速與 63.18 畫面更新/s（背景優化前 28.86×、62.91/s）；每個 Tick 約 .593 ms。最終封包 SHA-256 B00CAF0B4918C97A41A5B5568CA6F0DA0061BB1F6EC86D82ACD8F5E4F2DA6B4A，還原備份 `package.nw.before-layout-fixes` 是上一版巡邊。正式 Steam 安裝只置換 `package.nw`。

100× 容量對照：兩個 native clocks 實測都約 100×。用 stock 最大 dt 1/30，需要每實秒 3,000 次完整的循序 Tick；此機平均約 0.59ms/Tick（約 80% 在事件階段），相當於每實秒 1.77 秒的單線程工作，尚未計入繪圖。11ms 預算配 15ms callback 下限約 31–42× / 60–63 FPS；25ms 預算約 54× / 38–40 FPS；完全省略 Render 在較低回呼頻率仍只有約 55–57×。延長 dt 會改變 stock 離散碰撞與 `Every` 觸發，不是單純的效能修復。實測 WebAudio 在主線程忙碌時可能停頓後跳超過 500ms；若兩個 native clock 仍一致且依上一倍率推算沒有真停頓，不能把這個跳值直接當成背景中斷。

Upgrade 購買只在前端更新購買與完整樹，普通購買不直接存檔。#265 SID 399782854691094 每次刷新 83 個節點，#346 SID 609015691615495 為每節點用全部候選以距離 300 搜尋，等同最多約 6,889 次比較。確認座標真為唯一 300 px 方格才用每次刷新重新建立的索引和四個鄰居的目前 rank，否則回原條件。高倍率時約 60 ms 的真實慢影格會觸發舊排程 6000 ms reset，動畫 dt 變為 0；Upgrade 應先使用獨立的真實間隔並保留 stock dt cap。原生 100×、真正遊戲點擊、控制版加一次 ~99ms 人工 stall 後兩個取樣 Tick dt=0，修正版第一個取樣 Tick dt=1/30。連續五次購買後，比對 83 節點的等級、鄰居總和、解鎖和顯示完全一致，未重現長時間卡死，也未量到整體畫面 FPS 改善。

躲彈巡邊仍用 MoveTo 和 stock 碰撞、根據約 0.30 模擬秒的原速彈道及約 0.075 模擬秒的掃描間隔，綠色資源不當成威脅，密集彈幕有限制。隔離完整 NW.js、實際 Red 子彈 500 px/s、碰撞開啟：安全時維持貼邊，危險時偏離約 25 px，HP 維持 3，再回到巡邊。F9 與正常計分、結算無變更。

## 自動縮小與後兩種資源

自動縮小版本 SHA-256 1295C9AA5F9AA9E2A5A51B5D1C0F82CAD3031A964820E392E6542D7E603DD4FF；上一版 290B26F63FECFC13A7F2C3A7B02F516DDB6957893E0DDDF37E05EF135B9D7388 保存在 `package.nw.before-auto-shrink`。正式 Steam 重開後已比對解壓載入的 runtime 與封包完全一致。該次重開 OpenSpeedy 程序未執行、遊戲無原生注入；100× 驗證是在獨立原生副本完成，不能混稱正式遊戲已在 100× 運轉。

縮小沿用原版技能流程：僅對 `singleLeft` SID 608732816899725 和 `singleRight` SID 287501796200871 的 `CompareBoolVar` 加入合成 held 判斷，原手動布林為真時仍照常。需在 Dodge、存活、Shrink 群組有效、已裝備 shrink 且能量足夠，先嘗試正常移動，再確認縮小能避過預測碰撞才請求約 0.12 模擬秒，持續重評估。原版碰撞 #184 早於縮放 #239，所以預測納入兩個步進的啟動延遲；F8、死亡、場景與路線失效會解除。沒有直接寫尺寸、能量、血量或存檔。

真實完整 NW.js、碰撞開啟、相同四顆 Red 子彈的 1× A/B：舊版 HP 3→1，尺寸維持 75、能量 100；新版 HP 維持 3，尺寸 75→37.5→75，最低能量約 85 並回充。獨立 OpenSpeedy 原生 100× 的逐 Tick trace 也見相同啟動、解除與回充，180 步測試彈幕期間血量不變且無未處理錯誤；這是控制彈幕驗證，不保證所有密集彈幕都不受傷。

第四種橘色不是普通刷出或單純閃避所得。零橘色起步需先滿足原版前置條件，購買 `shield give orange`（100 紫色）後以 Shield 實際擋彈；`shrink give orange` 本身花 30 橘色，不能從零起步。第五種藍色的 Dodge 生成門檻為 `levelTime >= 150` 秒且存活，與 240 秒 Boss、300 秒 break box 無關。加速縮短真實等待時間，不降低這個關卡計時門檻。

## 依裝備自動使用四種技能

四技能版本 SHA-256 7BF4B4724051A7AACC7CFC036622A391BA3E235B078DDAF694E01074D7DC13F7；回滾 `package.nw.before-auto-skills` 是上述 1295C9AA 自動縮小版。沿用 singleLeft/singleRight 的原生按住輸入，僅自動使用目前裝備的兩種，換裝字串與 live player identity 防止沿用舊計畫。F8 控制自動移動和技能；F9 開自動刷分時同時開巡邊。原版 Upgrade #483/#489 在兩槽技能相同時清空另一槽，不需補丁支援重複裝備。

- 技能字串為 `shrink`、`slow`、`shield`、`shockwave`，各有獨立充能。Slow CurrentCharge SID 990876506082785、drain 985590669122070；Shield charge 468190106749655、drain 757548987014003；Shockwave charge 955318864596436、charging timer 708128531638183、charging duration 719031418869378。
- Shrink 仍以縮小後可通過的預測路徑為前提。Slow 保留原版 global 與玩家倍率差別；短請求自然到期，避免效果讓下一次掃描安全就立即開關。Shockwave 預設先花 1 次，持續按住 2.5 遊戲秒才清除半徑 400 內的 bullet，不能當作即時保命；附近至少三顆進入範圍的來彈才預先蓄力，已付費則持續至原版施放或合法失效。
- Shield 是階層子物件 SID 377362559195784 的方向扇形，MoveTo maxSpeed=0 只旋轉、不移動中心。自動瞄準僅替換 #284/#285 的唯一 aiming action，跟隨來彈方向。Zap 是獨立類別，Shield 不能擋，Shockwave 不能清，仍需移動閃避。
- 自動 held 並不重設實體滑鼠的 timeSinceClick。Shield 原版先收 drain×0.4 的預付費；若不處理，舊滑鼠計時會讓初始 0.4 秒額外持續扣款。只在原版 upfront TriggerOnce 成功的 #274/#279 記自動專屬 GetGameTime；#275/#280 唯一 ongoing drain action 在該預付窗內略過。其餘原版扣能／顯示／解除／回充照常；不寫真滑鼠計時器或充能。手動按住時不接管 aiming 或預付窗。

最小驗證：85 個獨立行為案例、封包 runtime 語法、ZIP 解壓後重建同 hash、testcopy upgrade/restore 還原同 hash。真實 native 100× 的前一候選完成 12 左右排列×gap/ring/safe 共 36 案例；最後僅修正 Shield 預付窗後，重驗 Shield+Slow／Shield+Shockwave 左右交換共 8 個 ring/safe 案例。實測 upfront 恰為 12、首 0.4 遊戲秒不再重扣、之後正常扣，解除回到 100；原版 shield 擋彈／銷毀 callback、shockwave #312 完成蓄力施放均有實際記錄。碰撞保持開啟，為觀察完整效果使用 fixture HP 100，不能宣稱正常血量存活率改善或完整實速 100×。正式 Steam 安裝重開已比對載入 runtime 完全相同、六程序官方 DLL 和 enable byte 均為 1，官方共享倍率前後不變。
