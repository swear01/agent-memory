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
