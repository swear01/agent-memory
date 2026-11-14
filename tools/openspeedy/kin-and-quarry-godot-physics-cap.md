---
title: Kin and Quarry：高倍數 OpenSpeedy 會碰到 Godot 的物理步數上限
scope: tools/openspeedy
status: active
updated: 2026-11-15
---

Steam 版 Kin and Quarry（App 3505770）的遊戲 log 確認使用 Godot 4.5.1。OpenSpeedy 選對程序且 speedpatch64.dll 已載入。用同一天挖礦的 TIME REMAINING 與牆鐘比對：2× 設定在大小視窗都實測約 2.0×；10× 設定在最大化約 3.7×、縮小後約 3.5×。短樣本只代表當時的遊戲場景。

遊戲 PCK 的 project.binary 設定 application/run/max_fps=60，未覆寫預設的 60 Hz 物理頻率或每幀最多 8 步。OpenSpeedy 加快 QPC，而 Godot 用 QPC 決定經過的時間；要補算的物理步超過上限時，Godot 會截短遊戲時間。按預設步數，外部 10× 需要 75 個實際畫面每秒；60 FPS 上限最多容許 8×，實際幀率更低時上限也更低。這是由設定、原始碼與實測推定，沒有單獨量到實際幀率或物理步數。

修正方式：在遊戲安裝目錄新增 Godot 的 override.cfg，透過 [autoload] 載入相容腳本。Steam 正常 Play 就會載入，不需改啟動選項、原本 PCK、遊戲執行檔或存檔。以 Godot 的 Time.get_ticks_usec（受 OpenSpeedy QPC 掛鉤）除以 Time.get_unix_time_from_system（Windows GetSystemTime，實測不受 OpenSpeedy 3.3.8 影響），每 0.35 真實秒估計使用者倍率。調整每個遊戲秒的物理步數到 round(120/倍率)，限制在 1 至原本的 60，適當提高單幀的步數上限，仍由 OpenSpeedy 決定遊戲時計。回 1× 時恢復原本物理步數。以 PID 掛鉤需在重開後重新勾選；OpenSpeedy 的 NAME 分頁可登記自動套用。

原版遊戲和 OpenSpeedy 3.3.8 的隔離實測，在 Level 內至少確認 0.5×、1×、2.5×、10×、25×、50×、100× 的引擎時間接近設定，10× 約 10.02×、100× 約 99.93×。100× 時每個物理步長達 0.5 遊戲秒，不保證碰撞、動畫和挖礦結果與正常速度等價；別宣稱 1000× 或以遊戲引擎時間等同挖礦效率。用同一局挖礦倒數驗證實際玩法，與引擎時間分開報告。

2026-11-15 修正畫面卡頓：OpenSpeedy 3.3.8 也縮短 sleep，使用 QPC 的 Godot 軟體幀率上限跟著放大；關 VSync 時，60 的上限在 10× 不再等於 60 真實 FPS。相容腳本在關 VSync 時以 round(使用者原本 FPS 上限/倍率) 補償，也尊重無上限、VSync 與遊戲選單更改；到 1× 還原。10× 名義 60 FPS 的引擎內部設 6，目標約 60 真實 FPS。Windows 個別遊戲或驅動設定不用改。

同一隔離礦場比較，限制約 60 真實 FPS 時，240 次物理更新/秒只跑 47–52 FPS，P95 25–34 ms；改 120 次後約 60 FPS、P95 約 18 ms，遊戲倒數仍為 10×。另用原存檔副本確認 10×/120 可走完一天，5.84 真實秒內倒數從 60 到 1.3，挖了 12 塊，角色有移動。全域插值在受控測試反而使 P95 變差，因此保留原本的關閉狀態。物理步減少會改變碰撞精度；不保證高倍率與原速相同。


- [Godot 4.5.1 frame loop](https://github.com/godotengine/godot/blob/4.5.1-stable/main/main.cpp#L4345-L4378)
- [Godot physics steps and time scale](https://docs.godotengine.org/en/stable/classes/class_engine.html#class-engine-property-max-physics-steps-per-frame)

2026-11-15 修正二十多倍的挖礦停擺：經過解碼與存檔副本驗證，miner/digger 的 MiningDelay 計時器會播放挖礦動畫，真正扣血由動畫 method track 觸發。兩者原來走畫面更新；若單幀遊戲時間大於短計時器週期，計時器可能對外顯示為已停，下一次物理更新就重啟計時器並取消尚未命中的揮擊。加速時把兩個節點改到物理更新，回原速還原。舊的 120 牆鐘物理步上限在 25× 壓到每遊戲秒 5 步，礦工位移能跳過 Area3D 接觸更新，實測有一個跌到 y=-399。新腳本保留每遊戲秒至少 12 步；原存檔副本五名礦工皆有移動與繼續挖礦。高倍率若物理跟不上，實際倒數速度會低於 OpenSpeedy 設定。

2026-11-15 修正降速卡死的一條路徑：用獨立的 3.3.8 speedpatch64.dll 和八條執行緒壓測反覆切換 25×/10×，多次觀察到同執行緒的 QPC 倒退，最壞約 100 ms 遊戲時間。這不等同重現正常切換下的整個遊戲卡死；Godot 4.5.1 Windows 軟體幀率限制使用 QPC 的絕對 deadline，延遲後甚至會忙等到到期。新腳本關掉這條軟體限制，改用未被 OpenSpeedy 掛鉤的 GetSystemTime 做有界等待，VSync 足以滿足幀率上限時交給 VSync。測試需用實際遊戲 Level、真的載入存檔（saves.load_game(false)，呼叫時 allow_saving=true，載入後再關掉存檔），並分開量 QPC 倍率、倒數、挖礦和切換後的真實 FPS。比對 3.3.8 到 3.3.19 的 speedpatch，QPC 換基準的程式無變動，不能把升版當成已驗證的修正。

- [Godot 4.5.1 Windows frame pacing](https://github.com/godotengine/godot/blob/4.5.1-stable/platform/windows/os_windows.cpp#L2576-L2639)
- [OpenSpeedy 3.3.8 QPC hook](https://github.com/game1024/OpenSpeedy/blob/3.3.8/src-bridge/speedpatch/speedpatch.cpp#L391-L421)

同一五礦工存檔在 1280×720、前向 Vulkan、請求 25× 的採礦對照：原畫面約 21–22 FPS，關日月陰影約 21 FPS（GPU primitive 與 draw call 約減半），單關粒子約 21.5 FPS，隱藏局部燈約 23 FPS。圖形效果會影響渲染負載，但當時主要限速並非它。高倍率自動關 Level 的陰影與泛光，回 10× 還原；不刪節點或停用劇情／挖礦腳本。在 2× 以上把單幀補算限制為 6 步，避免掉幀時一口氣補算二十步又擠掉下幀，速度不足時讓實際倒數自行降速。25× → 10× 測試應跨過至少一次場景分日，分開報告換日短暫停頓與是否恢復。

無滑鼠測試用 QUARRY_TEST_PROFILE 限定的 SceneTree 測試 API，只對白名單中的 BaseButton 呼叫 pressed.emit()，先驗證按鈕已啟用且可見；實測 start_day 執行真實處理器並進入挖礦。可使用 headless 的沙盒和複製的 AppData，不需切使用者焦點。OpenSpeedy 既有快捷鍵可透過 settings.json 的 gear4Shortcut/gear4Speed、gear5Shortcut/gear5Speed 儲存，程式啟動時才讀取並註冊；如果應用程式執行中寫檔，必須重新啟動才生效。

若在每輪倒數約 10 秒看見中央旋轉、大量粒子，Level 的 Geomancer 有六個各 100 顆、以 sin/cos 繞圈的發射器；最後一秒左右另有中央的 320 顆 MineResetParticle。只隱藏這些粒子及 Evertree 裝飾環的粒子、把 amount_ratio 設為 0，保留施法、重置和粒子節點腳本，減速後還原。screen_shake 是未被引用的設定；Block Shake 只旋轉個別方塊。本次另外在高倍率取消 Camera Auto-Pan 發起的新一天跳鏡，不覆寫玩家的存檔設定或手動鏡頭。

若晃動在倒數最後 11 秒之前也持續出現，EvertreeShaderRing 更符合：它的 shader 取樣 SCREEN_UV 並以正弦波位移三個色彩通道，所以鏡頭靜止也能讓背後景物抖動；同時畫移動星點並發出 100 顆粒子。高倍率只隱藏這個 MeshInstance3D 與粒子，保留 Evertree 本體和進度。

2026-11-15 後續在 Steam 正式程序，透過本機檔案信箱呼叫選擇存檔與「開始挖礦」，用同一天的兩個倒數樣本量有效倍率。移除 8,664 個礦塊空的 `_physics_process` 回呼後，25× 短樣本由約 16× 提到約 22×；倒數 41.6 秒切 10×，下一個樣本剩 27.2 秒、同一天再挖 15 塊、畫面多 56 幀，五人均移動。BlockPool 每次重生重新啟用回呼，於新挖礦日開始時對確切腳本且無 `_physics_process` 或 `_notification` 的礦塊做 `set_physics_process(false)`。隔離 A/B 在 9 Hz、4 worker 只關空回呼，25× 從 14.5× 到約 25×；SceneTree 每步中位數由 3.85 ms 降至 0.76 ms；逾八千塊的 RID space、collision layer、mask 和 shape disabled 逐一驗證不變。別用 PROCESS_MODE_DISABLED，這會影響碰撞。

藍色粒子是 Evertree 光環發射器，隱藏扭曲 mesh 並關發射；背景樹和灌木另有 wind shader，令 wind_strength=0，回 1× 恢復。原來超過 15× 才關光環，降到 10× 會重現。木塊震動是獨立的 block_shake 設定；實測存檔已關。6 步 × 45 FPS / 9 Hz 約只有 30×，提高每幀補算也會降低 FPS。隔離 60× 測試四執行緒約 46–47×，Godot 預設執行緒池約 49.5×，轉深層靜態碰撞體無改善；Godot time_scale=60 與 540 真實步/秒的對照約 37–40×，目前沒理由以它替換 OpenSpeedy。

Steam 40× 降到 10× 的換日確實卡死：兩份 minidump 的主執行緒均位於 Tween::step / PropertyTweener::step、_start_tweeners，起始 delta 為 2,305,843,003,392 秒，吻合 MainTimerSync 用 uint64_t 相減、再將下限設為輸入的八分之一；等 47 秒後仍追算。相容層在 process_frame 讀到異常 delta 會暫停正常節點，清掉僅用作裝飾的無限循環 Tween，下一個正常 frame 解除它自己設的暫停。用捕獲的異常值測試會略過該幀並保留有限 Tween 和既有暫停；不代表已修 OpenSpeedy 跨執行緒 QPC 競態。

- [Godot 4.5.1 unsigned frame delta](https://github.com/godotengine/godot/blob/4.5.1-stable/main/main_timer_sync.cpp)
- [Godot 4.5.1 Tween stepping](https://github.com/godotengine/godot/blob/4.5.1-stable/scene/animation/tween.cpp#L315-L409)

2026-11-15 根治降速再停擺：重現一次異常幀被 GDScript 防護攔下後，倒數與渲染仍進，後來兩天所有礦工卻在出生點完全不動。這道防護執行在 MainTimerSync 計算物理步數之後，不能修引擎已污染的計時歷史。以 OpenSpeedy 3.3.8 原碼重寫 Windows QPC detour，單一 SRWLock 同步上一筆實時間、累積虛時間、前一倍率和分數餘量，讓正常換速的 QPC 不倒退。保留共用成員在 8 位元組的 writable/shared PE section，否則位址雖一致而 DLL 跨程序設定無法傳遞。八緒擷取 4,140 萬筆、切速 2,581 次零倒退；分開程序測 1.75/25/10 讀到同樣倍率。用 Windows 使用者自己的平行安裝啟動 OpenSpeedy，保留原始系統版、沿用原設定，Steam 遊戲仍從原 App ID 啟動。正式接連四個真實挖礦日全部有六工人移動，25 設定實測 24.79，90 設定兩輪約 56.6–58.1 及 120 設定 42.8；原熱鍵在同一挖礦日 25→10，再挖 31 塊礦，畫面增 55 幀，沒有捕到異常幀。這套 QPC 修正未改所有其它 Windows 時鐘掛鉤。

隔離步數對照：90 設定 24 步比 12 步有效倍率更高，但渲染常只有 21–27 FPS、每局挖礦量降低。不要從倒數推論有挖礦；記錄同天兩個礦塊計數、各工人座標、實際畫面與異常幀。原版 damage queue 每三個渲染更新才處理 50 個，完全移到物理時間消耗會令挖得更多但 FPS 約減半；有積欠時補一批也未提高速度。單純跳過畫面或縮放解析度的對照沒有穩定可部署的改善。
