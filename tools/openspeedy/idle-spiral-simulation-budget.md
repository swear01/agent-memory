---
title: Idle Spiral 3.0.82 的外部倍速被模擬預算轉入離線記憶
scope: tools/openspeedy
status: active
updated: 2026-10-24
---

Idle Spiral 3.0.82 build 117，Unity 6000.5.7f1／Mono，搭配 OpenSpeedy 3.3.8。唯讀檢查加速 DLL、每程序啟用旗標與 shared 段倍率可證明注入設定，不能單獨證明玩法達完整倍速。

本機反編譯確認 `GameSystem.Update()` 以 `Time.realtimeSinceStartupAsDouble` 差值餵入 `FixedStepSimulationClock`。固定步長 0.02 秒，每個 rendered frame 最多 4 步；多餘完整步長由 `DrainRenderedFrame()` 轉為 offline seconds，`OfflineBoost.ApplyOfflineGap()` 每秒增加 10 單位記憶，不立即模擬戰鬥或生產。單次大於 2 秒的 elapsed gap 亦直接走此路徑。

外部獨立時鐘與自動存檔交叉量測：35.0394 真實秒，存檔程序時間戳增加 700.9469 秒（20.0045×），`dto.totalTime` 增加 195 秒（5.5652×），`offlineBoostAmount` 增加 5061.1495。`195 + 5061.1495 / 10` 與程序時鐘差約 0.168 秒，支持超額時間被轉存。第二段約 20.24 秒亦量得程序時鐘約 19.87×、遊戲時間約 4.55×。這些是場景與負載相關量測，不能當固定上限。內建 Offline Boost 在兩段皆關閉。

主資源、Battle Spiral、Equation、Reactor、Theory 等由 `IdleSpiralSystem.Advance(gameDelta, baseDelta)` 分派；戰鬥道具持續時間與部分 logger 使用 baseDelta。Equation 只有 `IsUnlocked && IsSelected` 才增加研究；沒有選取不等於加速失敗。自動購買／轉生每程序時鐘秒排程，並受每幀檢查限制。Daily Spiral 用 DateTime.Now 判斷日期，不能和主模擬時間混為一談。

內建倍率為 1 時，每個 rendered frame 最多直接模擬 0.08 遊戲秒；完整外部 20× 在正常路徑需約 250 個實際 rendered frames／真實秒。這是推算，非獨立 FPS 量測，不能把保存的 targetFrameRate 或被掛鉤時鐘顯示的 FPS 當真實吞吐量。

唯讀檢測可解析 `<persistentDataPath>/IdleSpiralSaveV2/shared-v1/primary.idlespiral-save`：外層 JSON 的 entries 各項以 key=dto/save/saveR 儲存內層 JSON 字串。用外部 Stopwatch、連續自動存檔及記憶量增量驗證，不需 UI。注意自動存檔延遲、totalTime 整秒量化與存檔重載，不能對沒有更新的檔案推斷零速度。

OpenSpeedy 3.3.8 的 `src-bridge/src/main.rs` 中 STATUS 直接回 OK，ISENABLED 也有 always-true 路徑；不能以該版 bridge 狀態回應代替唯讀啟用映射與進度量測。GETSPEED 可讀倍率，shared 段也可在確認 PE 段表後以 PROCESS_VM_READ 讀 double。勿把新版 bridge 行為外推到 3.3.8。

僅確認內部 C# 方法（Advance、TimeWarp、AdmittedSimulationDelta 等），沒有因此取得外部公開控制 API。TimeWarp 不涵蓋完整戰鬥等所有系統，不能當等價快轉。這次未做調倍率 A/B、FPS 優化或修改遊戲；改善方案仍須另測。

## 後續：執行中補丁已驗證約完整 20×

使用者後續明確授權插入補丁。先備份存檔與核心組件，建立本機獨立分支／worktree；以 Harmony 2.4.3（net472）在執行中的 `GameSystem.Update` 替換兩處時鐘呼叫，不改寫遊戲 DLL，也不操作 UI 或重開遊戲。直接 patch 該 Update 的呼叫位置，可避免只修改小型 clock 方法而受既有 JIT inline 影響。守衛包含已知 IdleSpiralDomain SHA256，以及 AddElapsed／DrainRenderedFrame 各恰有一處呼叫。

以已注入的 OpenSpeedy 模組之 `SP_IsEnabled`／`SP_GetSpeed` 讀取有效倍率。啟用且大於 1× 時，每幀預算為 ceil(4×factor)，20× 時 80 步；直接 Drain 預算步數，剩餘 backlog 保留，沒有再換成記憶。將大於 2 但不超過 2×factor 的 elapsed 分段餵給原 AddElapsed，使兩真實秒內的加速時鐘間隔不被誤判為離線。真正長間隔仍走原離線路徑；關閉或 1× 使用原邏輯，timeScale=0 仍不推進。

用同一份 Unity Mono 原生 DLL 建立獨立測試程序，先驗證完整步數、餘數、短／長 gap、停用／暫停、Harmony 安裝與卸除，再測 remote loader。全部通過後才注入遊戲。自製限定遊戲與測試 host 的 x64 loader 使用 Mono embedding API，逐 remote call attach／detach，64-bit exception 指標，超時保留執行中的配置；不使用新版未核實的 bridge STATUS，也不卸載仍可能被呼叫的 managed assembly。

同次套用前 20.2487 真實秒量得進度 4.988×、離線記憶增加 3159.2。首次套用後 30.3273 秒推進 606 遊戲秒（19.982×）、記憶增加 0；暫時卸除後 10.1444 秒回到 6.8018×、記憶增加 1463.8；重新套用且等待自動存檔穩定後，30.3519 秒推進 608 遊戲秒（20.0317×）、記憶增加 0。重新載入後立即取樣曾受舊自動存檔邊界影響，短樣本約 19.2×；不可隱藏這段或誤當穩定缺失。

核心遊戲檔案測前測後 SHA256 相同；沒有編輯或回滾存檔。補丁僅作用於當次程序，重開後須再載入。已驗證目前場景約 20×，不代表硬體可支撐任意倍率／後期負載，也不涵蓋伺服器端與所有獨立 UI／每日排程。

## 後續 v2：限制追算以防高倍率主執行緒飢餓

使用者另要求處理高倍率 freeze。第一版最高可排數千步／幀、積壓無界，存在大量追算風險；本次未重現歷史 freeze，不能宣稱找到所有成因。v2 替換 Update 中的完整模擬迴圈：最多 128 步，20× 仍為 80 步；每步才 Drain(1) 並走原本 TickManager／OfflineBoost／Advance，約每 8 真實毫秒於步驟邊界讓出。未執行時間與 Boost 不會預先消耗，兩個 admitted delta 對應實際執行步數。

持續過載的 backlog 上限為 min(40, 2×factor) 模擬秒，超出部分回傳原有 offline 路徑。因此極高倍率會犧牲實效速度、可能增加記憶，不再無限追算。這是 cooperative budget，無法中斷單一步驟、GC、繪圖或其他遊戲邏輯；實際觀察過單幀模擬約 19 ms，不能宣稱硬保證每幀 8 ms。

OpenSpeedy 3.3.8 掛鉤 QPC／tick／system clocks，不能用一般 Stopwatch 保護遊戲內的真實時間預算。改用 QueryUnbiasedInterruptTimePrecise（100 ns 單位）；本機實際 export 存在 KernelBase.dll 與 api-ms-win-core-realtime-l1-1-2.dll，Kernel32.dll 沒有，雖 Microsoft Learn 文件列 Kernel32.dll。隔離測試先遇到 EntryPointNotFoundException，確認 export 後改用 KernelBase.dll，再通過測試。勿只根據文件推斷該 Windows 組建 export。

Mono LoadFrom 可能重用同一 assembly identity，因此熱升級採不同 assembly name，先卸除舊 Harmony owner，再安裝新版本；若仍有舊版 owner，新版拒絕靜默略過。v2 同時驗證兩個核心 DLL 雜湊和迴圈 IL 結構。獨立 Mono 測試驗證 10,000 幀持續過載、限步、慢步驟 yield/resume、backlog 有界與模擬／離線／排隊總時間守恆，並完成 Harmony 安裝卸除。

實際升級後於原本 20× 設定，以外部 Stopwatch 觀察 30.3186 秒，遊戲進度增加 604 秒（19.9218×），記憶增量 0。31 次外部程序取樣全有回應，幀數與 yield counter 持續增加；是目前場景短期證據，不代表所有場景或歷史 freeze 都已消除。沒有對執行中的遊戲強行切換極端倍率，極端負載在隔離排程測試驗證。

## 各系統後續檢查：BigDTO 與日曆時鐘

讀取此版本存檔時，不能僅看 dto.r、dto.theoryDatas 等舊欄位：即使 IsBigDTOApplied=false，執行中的 Theory/Reactor 等原始碼直接使用 dto.bigDTO；實測 Theory 0 的 bigDTO.canProgress=true、exp +1639.899965 對應 +1640 遊戲秒，舊 dto.theoryDatas 卻全為 canProgress=false。從舊欄位推論「全部未研究」會錯誤。Equation 的 selected flags 與 Theory 研究不是同一系統。

本次各系統檢查時，外部倍率在觀察中升至約 102×（不是代理修改）。穩定高倍率的 40.3514 真實秒，程序時鐘約 102.199×、遊戲進度約 22.403×、離線記憶增加 32231.65；runtime backlog 接近 40 秒但有界、yield 持續增加、遊戲仍回應。這是限額保護介入的證據，不能宣稱完整百倍。R、擊殺、獎勵、Loot 與 Theory 均有進展。零投入 Reactor、未選 Equation、未解鎖 Design、未購買自動化、未開 Raid 等，只能標為條件未觸發，不能當 active path 已驗證。

已確認另一個缺口：DailySpiral 的預設 _clock 是 DateTime.Now，dailyLastResetDate 已超前真實日期一天；存檔 savedAtUtc 也約超前一天。EnsureCurrentDay 遇到儲存日期大於目前日曆時直接拒絕重置。因此外部倍速會污染日曆功能；v2 只保護模擬預算，尚未隔離日曆時間。不要擅自回改日期旗標、重發每日獎勵或回滾進度。舊日誌有 Analytics Curl timeout，不能以本機自動存檔正常推論雲端／線上功能成功。

## Steam 自動載入與成就查核

OpenSpeedy 3.3.8 的 Roaming 設定檔 `com.game1024.openspeedy/settings.json` 曾保存 speed=102.32929922807536，正好是 10^2.01，符合 SpeedPanel 對數滑桿 min=-3/max=3/step=0.01 的映射；只能證明保存值，不能推斷哪次操作改動。此版 bridge 的倍率為原生 DLL 共用區段，其他程序 LoadLibrary 同一 DLL 可讀取既有 factor；不要因加速器顯示值就判定實際模擬已達該倍速。

後續已建立可還原的 Steam 自動載入：在遊戲根目錄增加官方 UnityDoorstop 4.5.0 x64 winhttp.dll、doorstop_config.ini 與 IdleSpiralSpeedPatch payload，Steam 仍啟動原 IdleSpiral.exe，不改 Steam launch options 或原始遊戲組件。Doorstop entrypoint 僅依賴 mscorlib，透過 CREATE_NO_WINDOW 啟動 .NET 8 PatchLoader autoload helper，立即返回。Helper 等至少兩真實秒與本次 Player.log 的 `[mainthread] player loop thread=1` 標記，再使用既有 Mono embedding injector 呼叫 Loader.AutoInstall；90 真實秒仍未 ready 就記錄錯誤退出。先安裝 bounded simulation guard，再核對並載入本機既有 OpenSpeedy DLL，沒有寫入 shared factor 或設定檔。

獨立原生 Mono emulation 中，在 Doorstop 早期 entrypoint 直接載入完整 Harmony/game 相依組件曾原生崩潰；不能外推為所有 Unity/Doorstop 組合都不相容，也未完整證明單一 upstream 成因。最終改用最小 early bootstrap + 遊戲 ready 後的外部 helper，分別驗證官方原生 proxy -> bootstrap -> 正確 PID/runtime path 的 hidden helper，以及獨立 Mono host 的 AutoInstall、錯誤 native DLL hash 拒載、有效 DLL 啟用；WinHTTP 無網路 session forwarding 也通過。不要將被還原來源的較舊時間戳當作 MSBuild 一定會重編；測試 host 回復舊來源後曾因 incremental output 保留實驗程式，需 Rebuild。

自動載入 runtime 使用不同 assembly identity `IdleSpiral.SpeedPatch.SteamGuard`，避免沿用同程序中已載入的手動版 FreezeGuard。卸除原 owner 後，已在實際遊戲成功 AutoInstall、確認新 runtime frames/yield/admitted 持續增加、原核心組件 SHA256 不變、遊戲仍回應且套用期間無新增錯誤。安裝器具 ownership/hash manifest，遇既有 proxy/config/payload 不覆寫；實測重複安裝、異動檔拒絕、rollback 與未知檔保留。後續使用者自行重開，已補上真正 Steam 啟動的端到端驗證，詳下節。

成就的本機證據：SteamAchievement 由原 Update 判定條件，呼叫 SteamUserStats.SetAchievement / StoreStats；原程式未改，原 Steam 父程序與命令列保留，Player.log 有 `Received stats and achievements from Steam`。這支持流程仍連線及取到資料，不代表已實測新成就解鎖或上傳，不能保證全部成就。不得為驗證而直接修改成就旗標或強制解鎖。

## 真實日期、自動存檔與後續卡死

CalendarGuard 在四個已核對雜湊的組件（Assembly-CSharp、IdleSpiralDomain、IdleLibrary、IdleSpiral.CloudSave）中，掃描並 transpile 53 個直接讀 DateTime.Now/UtcNow 或 DateTimeOffset.Now/UtcNow 的方法，包含 iterator/lambda。只處理主遊戲與 Domain 會漏掉外層 PersistenceV2 savedAtUtc、備份及 season deadline。使用 GetSystemTime 取得 UTC、需要時轉本地時間；此版 OpenSpeedy 沒有掛鉤它。獨立 Mono 掛鉤測試確認全球 DateTime 前進約 55.009 秒時，替換後日期只前進一真實秒，既有已編譯每日 clock delegate 亦受修正。保留原先未來 dailyLastResetDate／已用次數，不回滾或重發獎勵。第三方 SDK／伺服器未全面覆蓋。

Harmony 掃描 generic method body 時，無 generator 的 GetOriginalInstructions／ReadMethodBody 曾因 MonoMod ImportGenericParameter 擲 NotSupportedException；傳入 dummy DynamicMethod.GetILGenerator 後可讀取所有 body，再針對實際含日曆呼叫的 generic 方法拒絕，已知版本候選皆非 generic。

原自動存檔每 15 個程序秒排程，高倍率下實測八真實秒寫 21 次。只對 saveCtrl.BeginAutomaticSave 加獨立真實時間 gate：每個 controller 首次允許，成功開始後至少 15 真實秒再允許；被原程式拒絕的嘗試不推進 gate。保留背景準備／atomic write／completion，SaveImmediately 的手動、失焦、暫停及離開存檔沒有頻率 gate。實測保存間隔 15.047、15.154 真實秒，內外層時間戳接近外部 UTC。

外部存檔 observer 必須排除開始前已存在的第一份存檔，再比較至少兩次新寫入；15 秒舊存檔會讓短樣本倍率嚴重失真。預設窗口改 40 秒。CalendarGuard 下 ProcessClockMultiplier 僅代表存檔時間戳速率，不能拿來檢查全域 QPC 是否仍被加速。

後續真實 Steam 重開已確認原 exe、Steam 父程序、當次程序自動載入、53 處日期修正與 bounded simulation 都成功；10× 下 30.308 真實秒前進 304 遊戲秒（10.0304×），離線記憶增量零。新局再次讀取 Steam 成就成功，但新解鎖／上傳仍未測試。

約 55× 的舊局仍出現 Windows AppHangB1／1002 並由使用者重開，沒有留下當時傾印，原因未定；不能將此前幾十秒回應正常、autosave 修復或 v2 預算測試當成已修好高倍率 freeze。可用本機 ProcDump `-h -ma -r -at 5 -n 1 <pid> <local-dump-folder>` 等下一次持續無回應，捕獲一次或遊戲退出就結束，不殺程序、不上傳、不設排程。此案例擷取仍待發生，不能宣稱已拿到或分析堆疊。

升級安裝器先驗 ownership／全部既有 hashes／四遊戲組件，僅原子更新 CalendarGuard 與 Bootstrap，鎖檔失敗會回復先前已更新檔與 manifest。PowerShell 呼叫 File.Replace 的 null backup path 要傳 `[NullString]::Value`；`$null` 被轉為空字串會擲 PathEmpty。已有 CalendarGuard 的同 identity 熱換版須先關遊戲；首次新增可另以現有 injector 套用，不自行重開使用者遊戲。

OpenSpeedy 3.3.8 的 useSpeed 會在 mount 時將保存值送回 bridge；不能只因重開後倍率不同就推斷使用者改值。設定檔 mtime／hash 可證明前後異動，無法識別 writer。補丁沿用 SP_GetSpeed 且無 SP_SetSpeed，亦不能單憑此斷言外部設定變更的成因。

## Google 雲端檢查：JWT 已過期但本機日期仍在未來

Idle Spiral 3.0.82 build 117 的已連結帳號可能顯示 Cloud check failed，Player.log 記錄 storage 403。此案例真實 UTC 約 16:49，JWT exp 已於 15:42 過期，但 AuthToken.ExpirationTime 被加速時鐘保存成隔日 08:14；SDK IsValidWithMargin 誤判有效，EnsureValidTokenAsync 因此略過更新。舊 Player-prev.log 的 OAuth timeout 不是這次畫面失敗的直接原因，必須對齊程序與目前錯誤。

AuthGuard 對 IdleSystem.Auth 與 IdleSystem.CloudSave 兩個 hash-locked SDK 的四個方法加 Harmony transpiler：AuthToken.IsValidWithMargin、FirebaseAuthService.SetupTokenRefreshTimer／ConvertToAuthToken、Core.Save.CloudSaveManager.ApplyToken。五個 UtcNow 呼叫改讀 GetSystemTime 真實 UTC，兩個 ExpirationTime getter 改取 min(原本本機 expiry, JWT exp)。解析 exp 只決定是否請原 SDK 更新，不驗證簽章、不接受身分、不繞過伺服器；保留原五分鐘 margin。無效、缺少、非整數或溢位 exp 視為到期。ConditionalWeakTable 每個 token 快取解析結果，IdToken 改變即失效，避免每次 UI 判斷都解析 JSON。

隔離 Mono 測試通過過期 JWT 搭配未來快取、有效 token、token 變更、較早本機 expiry、margin 與無效 exp。實際遊戲透過下一次 GameSystem.Update 的一次性 postfix 在 Unity 主執行緒呼叫原 EnsureValidTokenAsync，更新成功且帳號相同。原 CrossPlatformCloudSaveCoordinator.InspectAsync 成功回傳 LocalOnly，使用者按 RETRY 後確認顯示尚無雲端備份，沒有登出或任何雲端寫入。診斷只輸出日期與布林結果，禁止輸出 JWT、refresh token、UserId 或 email。

唯讀雲端檢查必須使用這個遊戲實際採用的 VfsBundleCloudSaveService 及原 coordinator；BundleNotFound 在無備份帳號會由原 coordinator 正常轉為 LocalOnly。不要拿未使用的 RevisionCloudSaveService endpoint 失敗推論目前雲端仍壞。也不要以 generic VfsCloudSaveService.DownloadAsync 當唯讀替代，它會執行 CleanupOrphansAsync，可能刪除遠端孤立檔。

Steam bootstrap 改進入 AuthGuard，先安裝 CalendarGuard、AuthGuard，再呼叫原模擬 autoloader；仍不寫倍率。安裝器現在驗六個遊戲組件、可回復鎖檔失敗，已有 guard 同 identity 熱換版需先關遊戲。新增 AuthGuard 的原生啟動入口及隔離 autoload 已通過，但這版之後的完整 Steam 重開尚未實測，不可沿用先前 CalendarGuard 的重開結果宣稱新鏈已完整驗證。本機來源提交 d90dec7。

## Battle Spiral：先量戰鬥時間，再分辨 Nuke 模式與吞吐量

BattleCtrl.UpdateBattle(gameDelta) 由 IdleSpiralSystem.Advance 呼叫，與主模擬同一 fixed-step 路徑。一次 30.39 真實秒的主執行緒 observer 量到主模擬／戰鬥均約 943.12 秒（31.04×），而 OpenSpeedy 設為 41.69×；戰鬥並沒有單獨漏加速。排隊近 40 秒、8 ms budget 持續 yield 時，設定倍率不能等同實際倍率。

PerformanceBattleProgress.Modifier = 1 / (reward.GetRewards().Count() * skipNum)。Nuke 10 若有 21 格獎勵，攻擊進度就是 1/210，原遊戲以變慢出手換批次獎勵／進度。必須讀目前 CurrentNukeMode、modifier、AllyStatsManager.GetSecondPerAttack，不能把長攻擊條直接判成倍速失效，也不能擅自移除這個平衡機制。

實測 attack scheduler 的 Accumulate 實際參數加總，滿足「產生進度＋初始小數＝直接攻擊次數＋末尾小數」，誤差小於 1e-6；pending=0，敵方更新收到相同 gameDelta。沒有擊敗敵人的窗口只能確認時鐘、出手與傷害，不可宣稱獎勵結算已實測。畫面還有 BattleUiRefreshGate／DamagePopupBuffer，動畫或彈字數不等於真實攻擊數。一次性 observer 使用獨立 Harmony owner、獨立真實時鐘、Unity 主執行緒讀取；30 秒後移除自己，再明確 Remove 確認，禁止納入自動載入。

## 後續 AdaptiveGuard：解除固定 8 ms 的額外吞吐量限制

當設定約 41.69×、v2 固定 8 ms 時，一段獨立真實幀觀察量得 36.71×／43.27 FPS，先前另一窗口約 31×。獨立 AdaptiveGuard 只對已知 v2 runtime SHA-256 DBFEEBDC66091F27F63320971CD3F369B3FD66928FC6A66AF336425E44DF635F 的 RunFrame 中唯一 80000 tick 常數做 transpiler，另以 prefix/postfix 觀察 RunSimulationFrame。可用新 Harmony owner 單獨移除，保留原 v2、日曆與登入修正，避免同 assembly identity 的熱換問題。原模擬 DLL 本身不重編。

預算在 8–16 ms 間依 backlog 升降（每幀 +0.25／-0.125 ms），以外部工作耗時 EMA 限制相對 40 ms 的軟幀目標；基準仍至少 8 ms，保留原每幀 128 步及最多 40 遊戲秒 backlog。仍不能中斷單一步驟／GC，不代表已解決歷史高倍率 hang。兩次直接觀察 41.6644×、41.6648×，FPS 為 31.58、34.76；安靜時外部存檔觀察 41.7686×、offline memory 增量零。另一段與工具建置測試重疊窗口只有 28.8113×且有 offline overflow；必須保留此失敗窗口，不可宣稱所有負載都達滿倍率。較多模擬 CPU 時間會犧牲一些畫面幀率。

原 v2 patch-status.txt 仍印出 baseline frameBudgetMilliseconds=8，實際動態值以 adaptive-status.txt 為準；原 duration／backlog 計數仍有效。不要把跨越清積壓階段、含過時自動存檔或重建工具的速度當穩態。FrameProbe 以獨立 QueryUnbiasedInterruptTimePrecise 計真實 frame period／GameSystem.Update duration，15 秒後自動移除，完成後另 Remove 確認；不放進 autoload。

隔離測試通過 16 ms 慢步驟讓出與恢復、一萬幀過載時間守恆、8–16 ms 邊界／headroom／1×、own owner 移除；原生 Bootstrap 與完整隔離 Adaptive→Auth→Calendar→v2/OpenSpeedy 載入鏈也通過。新安裝器另驗 baseline runtime hash，拒絕不支援的既有／打包 runtime，並保有鎖檔回復與 ownership。真實遊戲已套用且下次 Steam 啟動入口已安裝，但此最新版的完整 Steam 重開仍待驗證。本機提交 4e49ce3。

## 主螺旋顯示與 Theory 計算

遊戲已有 `CreateSpirals.SetRenderingSuppressed(object owner, bool)`，以 owner 集合管理主螺旋渲染抑制。用單獨 owner、在 GameSystem.Update postfix 的 Unity 主執行緒呼叫，移除時只釋放自己的 owner。保留 LineDraw 的角度與歷史更新，不能為了關 3D 而停用整個 LineDraw／Spiral：A_TotalTheta.Start 訂閱 Spiral.ThetaChanged，NotifyAchievements 依角度更新成就。此接口只隱藏主螺旋，不能宣稱所有戰鬥／方程式預覽都被關閉。當次場景沒有驗證到 FPS 收益。Install 必須先設定 requested 再安裝 postfix，Remove 一律排到主執行緒釋放並 unpatch，避免安裝／移除競態。

TheoryContainer.UpdateTheories 內原版逐項更新 EXP，但同次更新的兩個共用進度參數不變。只改寫 TheoryContainer 外層編譯器產生的 `<ApplyProduceLogic>b__19_1`／`b__19_2`，以 ThreadStatic cache 在該次 UpdateTheories 內惰性計算 study_progress／study_progress_add；finalizer 清理，巢狀呼叫回原 getter，下一模擬步重新求值。不可全域快取 Parameter.Number，生產參數包含週期及即時狀態。

隔離測試 10,000 步／100,000 筆 EXP 的 mantissa bits 與 exponent 均與原版一致，涵蓋變動共用參數、等級、材料、停用第一項研究、混合步長與例外清理。當次 live 抽樣 Theory 每步約 56.60→10.98 µs，整步約 212.37→173.17 µs；這是有觀測成本的場景結果，不保證全場同等提升。以每 127 步取樣避免與 CurrencyUpdateStepCount=4 同相；128 步取樣會產生嚴重偏差。巢狀階段耗時為 inclusive，禁止直接相加。

純 Formula cache 隔離測試成功，但 live cache-eligible hit／miss 都是零，沒有可驗證收益，已移除且不納入 payload／autoload。便宜分支繞過計數，不能說「函式呼叫為零」。保留實驗失敗證據，不應為有寫程式就上線。

新版啟動鏈為 Theory→Render→Adaptive→Auth→Calendar→v2/OpenSpeedy；完整隔離載入、原生入口、升級／鎖檔回復通過，但真正 Steam 重開仍待驗證。同 assembly identity 在 Mono 已載入後不可直接覆寫並假定熱換生效，需區分 live 與下次啟動 binary 的 SHA-256。

## 數秒一幀的原生停頓：仍未確定根因

同次觀察後期出現約 5～7 秒一幀；GameSystem.Update 平均數毫秒、其外部耗時數秒。撤下 Render／Theory 後仍約七秒一幀。暫時解除引擎 FPS cap 無效且已還原；短暫停用 OpenSpeedy 約二十秒未恢復更新，已還原啟用狀態與原倍率，不能由此排除已累積的加速時鐘問題。兩份本機小型 dump 的主執行緒分別位於 steamclient64 與含 Steam 程式庫指標的原生等待；NVIDIA worker 堆疊指標也經過 speedpatch64。僅指標掃描不等同完整 unwind，不足以確定根因，不能宣稱遊戲 freeze 修好或穩定 200–500×。

ProcDump PSS clone 完整傾印曾因驅動錯誤失敗，錯誤含 target no longer running，但原 PID 其實仍活著；不可只憑診斷訊息宣稱遊戲已退出。之後直接 `procdump -mm` 成功取得約 47 MB 小型 dump，約暫停 0.8 秒。dump 可能含登入資料，只留本機，不分享／上傳；套件僅保留不含秘密的效能與 hook 清理證據。最後以 Harmony.GetAllPatchedMethods 唯讀稽核，確認 observer 與 formula owner 為零。
