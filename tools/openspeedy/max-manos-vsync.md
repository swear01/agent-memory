---
title: Max Manos 的 OpenSpeedy 加速受 VSync 限制
scope: tools/openspeedy
status: active
updated: 2026-10-12
---

Windows Steam 版 Max Manos 搭配 OpenSpeedy 3.3.8，使用者回報倍速沒有作用。本機唯讀檢查確認遊戲載入 speedpatch64.dll、每進程啟用旗標為 true、共享倍率為 10.0，且遊戲匯入的 QueryPerformanceCounter、Sleep、timeGetTime 等入口已被掛鉤。存檔設定 optVSync 為 1。

請使用者在遊戲視覺設定關閉 VSync 後比較不同倍率。使用者最初回報沒有變化，隨即更正為「有變化了」。因此此案例已驗證關閉 VSync 能讓倍速產生效果；沒有量測實際倍速精度，不應宣稱已達完整 10 倍。

遇到同遊戲同症狀，先檢查 VSync，再調查注入、權限或位元數。GameMaker 支援以垂直同步作為渲染計時基準，而 OpenSpeedy 修改 Windows 計時與等待 API；垂直同步限制是此案例的合理機制解釋，但未對引擎內部做完整追蹤。

## 高倍率平台期：按幀玩法受處理吞吐量限制

同日後續排查，使用者確認 5×、10×、20× 看起來差不多。唯讀解析 data.win 的 GEN8 確認基礎速度為 60 FPS；解析 VARI、FUNC 與 CODE 後，確認 objCTRLgame 的 Step 事件對 runTimer 做一次條件式扣減，並以 runTimer mod 60 檢查秒邊界。因此倒計時依賴實際完成的 Step 數；完整 10× 需要約 600 Step/秒。

在共享倍率維持 10.0 時，用未受遊戲 speedpatch 影響的外部牆鐘讀取引擎幀計數器，10 個約一秒的區間量得約 100.7～132.9 次更新/秒，對應基礎速度約 1.68～2.22×。另一段狀態較輕的取樣中，引擎 fps 統計回到虛擬時間下的約 58～60，delta_time 約 16680 微秒；與重負載下約 74000～97000 微秒形成明顯差異。這支持場景負載造成平台期，不能宣稱 GameMaker 有通用固定 5× 上限。主執行緒另一次 8 秒採樣消耗約 7.45 秒 CPU；但 CPU 忙碌也可能包含引擎忙等，不能單憑此數字把所有耗時歸因於遊戲邏輯。

OpenSpeedy 加速後，引擎 fps、fps_real、delta_time 可能使用被修改的時鐘，不能直接當作真實牆鐘下的 FPS。診斷時應使用外部牆鐘和幀計數，或明確校正虛擬時鐘。此案例沒有進行 CPU/GPU 分項性能剖析，未驗證降低畫質的收益。

PresentMon CLI 在一般使用者權限下回報 failed to start trace session: access denied，需要管理員或 Performance Log Users。本次改用只讀引擎統計而未提升權限；後續補測時遊戲進程已退出，不能把退出後的零 CPU 樣本當有效數據。

## 輕負載實測接近完整 10×

再次啟動同版遊戲並由 OpenSpeedy UI 對新進程啟用 10×，在升級選單直接計數約 590.7～593.5 Step/秒；在只有少量方塊、畫面顯示 Click anywhere to start 且倒計時未開始的場景，10 秒直接計數約 584.8～596.9 Step/秒（約 9.75～9.95×）。這是輕負載狀態，不能外推到連鎖爆炸多的實際戰局，也不能把這次啟動與前次低速狀態的差異宣稱為設定優化的收益。

只讀檢查 funcGAMEnumbersDisplay 確認 optNumbers 為 false 時會跳過其數字特效處理分支；因此 Show Numbers 是有程式依據的減少工作量選項，但尚無慢速同場景下的開關 A/B 性能數據。優化需要先重現負載，不能用已達目標幀率的等待場景宣稱突破。

## 後續實測：設定調整未突破高負載上限

使用者授權自行開遊戲後，以備份存檔建立暫時的合成負載：300 個 Square、300 個 Triangle，MoreTime1 為 20（畫面倒計時 40 秒）。這不是使用者原本的後期戰局，也沒有重現高階連鎖技能；不可外推為所有戰局的速度。

每組使用外部牆鐘取樣約 8 秒：全螢幕、VSync Off、10×，等待點擊開局時約 255～259 Step/秒；開始實戰且游標固定在中央時，Show Numbers On 約 198、Off 約 206 Step/秒。後者只有約 4% 差異，因隨機位置與測試間等級變動，尚不能宣稱穩定收益。SleepMargin=0 且 Numbers Off 時，等待約 259、實戰約 207 Step/秒，與原本 SleepMargin=10 接近，沒有大幅改善。等待場景另測 5× 約 254、100× 約 252 Step/秒，持續提高倍率無法提高處理吞吐量。

同次實戰取樣 7.74 秒，最忙執行緒使用 7.67 秒 CPU，NVIDIA GPU 整體使用率 29～40%；結合等待設定對照，支持主執行緒吞吐量為主要瓶頸。仍未進行函式級 profiler，不能把耗時細分為 VM、碰撞、繪圖提交或掛鉤開銷。Ryzen 7 3700X、RTX 3060；Windows Balanced 電源方案。

此機器把 Fullscreen 關閉成預設較小視窗後，即使 VSync Off、倍率 10×，等待場景降至約 60 Step/秒。已觀察到視窗路徑的額外限制，但尚未證實是 DWM、驅動設定或 runner 的哪一層，不應把縮小視窗泛稱有效優化。

存檔隔離注意：funcCOREloadSavedGame 優先經 steam_file_exists/steam_file_read，再用 ini_open_from_string 載入；只修改 LocalAppData 的 maxmanos.ini 不能保證下一次讀取該版本。funcCOREsaveGame 會透過 steam_file_write_file 寫回 Steam Remote Storage，測試後須還原本機與 Steam userdata/<account>/<appid>/remote/maxmanos.ini，不能只還原其中之一。保存最初的乾淨備份；開始測試後再備份 Steam 快取，可能已含測試資料。

## 補測戰鬥中的 100×

同樣 600 個基本圖形、全螢幕、VSync Off、Numbers On、SleepMargin=10，外部牆鐘各取樣約 8 秒：10× 實戰平均約 202.85 Step/秒（3.38×），100× 約 194.57 Step/秒（3.24×）。每個倍率只有一個有效樣本，位置隨機且等級由 0 變 1，不可據此宣稱 100× 必然較慢；只能說此合成負載沒有顯示提高到 100× 帶來明顯吞吐量收益。100× 回合的倒計時也在約 8 秒真實時間內減少約 29 秒，與每秒更新取樣相符。

本輪 Alt+Enter 往返顯示模式後，即使 Fullscreen On / VSync Off，仍重現約 60 Step/秒；VSync On/Off 重新套用沒有解除，重開遊戲後恢復等待場景約 260、實戰約 203 Step/秒。此受顯示狀態限制的初步樣本已排除。切換焦點後的畫面尺寸可能尚未恢復，先等顯示穩定並重新讀取視窗，不要急著切換顯示模式。

## Draw 跳幀實驗：部分改善，仍未達 10×

以 UndertaleModTool CLI 0.9.2.0 建立暫時測試版本。FingerGun 模式的 crosshairX/Y/Dir 更新實際寫在 objCTRLgame 的 Draw GUI End（Draw_75）；直接停用全部 Draw 會改變玩法。因此先把原有瞄準計算移至 End Step，再對同一測試版本比較正常繪圖、每 5 次更新繪圖一次、每 10 次更新繪圖一次。只在未暫停的實戰階段使用 draw_enable_drawevent 跳幀，選單、等待開局與暫停維持正常繪圖。

仍用 300 Square + 300 Triangle 合成負載，MoreTime1 暫設 5980（6000 秒回合）以便同一戰局反覆切換。全螢幕、VSync Off、Numbers On、SleepMargin=10、OpenSpeedy=10×，外部牆鐘每模式兩次 8 秒取樣，順序 1/5/10/1/5/10：正常繪圖約 194.73、203.85 Step/秒；stride 5 約 252.95、248.81；stride 10 約 260.84、260.59。均值約 199.29 → 250.88（+25.9%）→ 260.71（+30.8%），對應約 3.32× → 4.18× → 4.35×。按步數與 stride 推算，後兩者每秒繪圖約 50 與 26 次；不是獨立量測螢幕呈現 FPS。

F7 診斷記錄核對：三種 stride 下 runTimer 扣減量等於完成的更新次數，繪圖次數符合 1/N；stride 10 移動游標後瞄準方向改變，擊破數由 240 增至 316、金錢由 560 增至 840。暫停期間 3265 次控制器更新中 runTimer 保持 330008，恢復後繼續扣減。手動結束回合顯示擊破 412、收集 1320、倍率 1.6、所得 2112，返回商店及寫入存檔符合結果。

這是基本操作驗證，不能證明所有模式與升級的玩法完全等價；未重現使用者的後期連鎖爆炸，沒有確定性重播或函式級 profiler。可確認降低 Draw 工作量有部分收益，不能宣稱已突破至 10×，也不能把剩餘耗時全歸因於碰撞或 VM。測試補丁未永久安裝，原版 data.win 與本機及 Steam 快取存檔皆以測前備份還原並核對 SHA256。
